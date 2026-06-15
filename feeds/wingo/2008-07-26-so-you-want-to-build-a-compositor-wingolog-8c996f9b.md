---
title: so you want to build a compositor — wingolog
url: https://wingolog.org/archives/2008/07/26/so-you-want-to-build-a-compositor
published: "2008-07-26T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/07/26/so-you-want-to-build-a-compositor
---

# so you want to build a compositor — wingolog

## [so you want to build a compositor](/archives/2008/07/26/so-you-want-to-build-a-compositor)

26 July 2008 10:02 PM

- [nargery](/tags/nargery)
- [gl](/tags/gl)
- [tfp](/tags/tfp)
- [window manager](/tags/window%20manager)
- [clutter](/tags/clutter)

**a dialog with someone i don't know**

> One way to think about it is that it's like driving a car down the road, and suddenly swapping the steering wheel and brakes out for a tiller and gear shifter. And having to downshift for braking until you learn that the brakes moved to the turn indicator lever. By trial and error.

That's the internet's Joey Hess, a wonderful writer, [who recently switched window managers](http://joey.kitenet.net/blog/entry/awesome/).

A while back I [discussed some new interaction possibilities](//wingolog.org/archives/2008/06/10/regarding-decadence), and a lot of them hinged on the relation between people and the set of tasks that they do on the computer -- not just the tasks themselves, but the relationships between the tasks, and with focus.

In the free desktop, the locus of this experience is in the window manager. Again, Joey tries to give analogies:

> Another way to look at it is adopting a new philosophy. Or, in some cases a cult. (In some cases, with crazy cult leaders.) Whether they use Windows or a Mac, or Linux, most computer users are members of a big established religion, with some implicit assumptions, like "thy windows shall be overlapping, like papers on the desktop, and thou shalt move them with thy mouse".

The obvious step if you want to [apostatize](http://www.apostasia.es/) yourself from the "desktop metaphor", then, is to start hacking window managers. That's how I spent my hack-time in the last month and a half; this writing is an attempt at exegesis.

**descent from the mountain, source in hand**

I wanted to make a 3D compositing window manager. By 3D, I mean that I wanted the pixels behind the monitor glass to come entirely from one process' OpenGL. By "compositing", I mean that the graphical output from other programs should be redirected through my program. ( [Compositors](http://en.wikipedia.org/wiki/Compositing_window_manager) don't necessarily need to be implemented with OpenGL; xcompmgr and metacity's compositor are implemented with XRender.)

As an implementation strategy, I took this opportunity to check out [Clutter](http://clutter-project.org/), a GL-based canvas library. What follows is a minimal translation into C of the basic concepts, a broken but fun-to-hack substrate for experimentation.

```
int main (int argc, char *argv[])
{
    prep_clutter (&argc, &argv);
    prep_root ();
    prep_overlay ();
    prep_stage ();
    prep_input ();

    g_main_loop_run (g_main_loop_new (NULL, FALSE));

    return 0;
}

```

The main function is pretty simple. We'll be looking at the various functions one by one, but out of order. Let's start with `prep_root`:

```
Display *dpy;
Window root;
void prep_root (void)
{
    dpy = clutter_x11_get_default_display ();
    root = DefaultRootWindow (dpy);

    XCompositeRedirectSubwindows (dpy, root, CompositeRedirectAutomatic);
    XSelectInput (dpy, root, SubstructureNotifyMask);
}

```

This function does two things of interest: first, it tells the X server to redirect output of all windows into backing pixmaps, such that the contents of all mapped windows are available, even if the X server does not think that they are visible. Second, we ask the X server to give us notification when windows come and go, and when they change size.

```
Window overlay;
void prep_overlay (void)
{
    overlay = XCompositeGetOverlayWindow (dpy, root);
    allow_input_passthrough (overlay);
}

```

The Composite extension, which allows us to redirect window output, also provides for the existence of an "overlay" window, one that is above all other windows. You can draw directly to the overlay window, but the normal way that you use it is as a parent window -- you create your own window, then reparent it to the overlay.

Composite is pretty cool, except that it is only for output -- that is, you can put the output of a window anywhere, like on the corner of a [Compiz](http://en.wikipedia.org/wiki/Compiz) cube, but you can't redirect input. By that I mean to say that when you have a rotated compiz cube, you can't interact with the window.

The reason for this is that in order to interact with a window, for example to click a button in it, the X server needs to know how to map the pointer position to a position inside a certain window. [This is a tricky problem](http://lists.freedesktop.org/archives/xorg/2007-February/021515.html) which needs to be done inside the X server, and no solution to this problem has been merged yet.

This upshot is that what happens today with compositing window managers is that the compositor is very careful to make sure that the underlying X window is exactly positioned behind where the compositor is drawing it. Then, when the user clicks on what essentially is the *redirected image* of the button, the click actually falls behind the overlay to the actual X window.

Hence, the need to allow events (clicks, pointer motion, etc) to pass through the overlay:

```
void allow_input_passthrough (Window w)
{
    XserverRegion region = XFixesCreateRegion (dpy, NULL, 0);

    XFixesSetWindowShapeRegion (dpy, w, ShapeBounding, 0, 0, 0);
    XFixesSetWindowShapeRegion (dpy, w, ShapeInput, 0, 0, region);

    XFixesDestroyRegion (dpy, region);
}

```

Basically we call a few undocumented functions, whose goal is to tell the X server that the window in question is not to receive pointer input events.

```
ClutterActor *stage;
Window stage_win;
void prep_stage (void)
{
    ClutterColor color;

    stage = clutter_stage_get_default ();
    clutter_stage_fullscreen (CLUTTER_STAGE (stage));
    stage_win = clutter_x11_get_stage_window (CLUTTER_STAGE (stage));
    clutter_color_parse ("DarkSlateGrey", &color);
    clutter_stage_set_color (CLUTTER_STAGE (stage), &color);

    XReparentWindow (dpy, stage_win, overlay, 0, 0);
    XSelectInput (dpy, stage_win, ExposureMask);
    allow_input_passthrough (stage_win);

    clutter_actor_show_all (stage);
}

```

The next step is to create the clutter "stage", the GLX window to which all of our output will go. We reparent the stage to the overlay, so that it will always be on top, then we allow input events to pass through it as well.

```
Window input;
void prep_input (void)
{
    XWindowAttributes attr;

    XGetWindowAttributes (dpy, root, &attr);
    input = XCreateWindow (dpy, root,
                           0, 0,  /* x, y */
                           attr.width, attr.height,
                           0, 0, /* border width, depth */
                           InputOnly, DefaultVisual (dpy, 0), 0, NULL);
    XSelectInput (dpy, input,
                  StructureNotifyMask | FocusChangeMask | PointerMotionMask
                  | KeyPressMask | KeyReleaseMask | ButtonPressMask
                  | ButtonReleaseMask | PropertyChangeMask);
    XMapWindow (dpy, input);
    XSetInputFocus (dpy, input, RevertToPointerRoot, CurrentTime);

    attach_event_source ();
}

```

So if the events fall through the stage, and fall through the overlay, what happens if they don't fall onto a window? Well, you probably want the stage to get the last crack at them, instead of the root window. So hence this terrible trick, making a separate fullscreen input-only window, located below all windows except the root window. We select for all input on this window, so that we get e.g. key presses as well.

Then we install a [GSource](http://library.gnome.org/devel/glib/unstable/glib-The-Main-Event-Loop.html#GSource) to process pending X events, redirecting events from the input window to the stage window:

```
GPollFD event_poll_fd;
static GSourceFuncs event_funcs = {
    event_prepare,
    event_check,
    event_dispatch,
    NULL
};
void attach_event_source (void)
{
    GSource *source;

    source = g_source_new (&event_funcs, sizeof (GSource));

    event_poll_fd.fd = ConnectionNumber (dpy);
    event_poll_fd.events = G_IO_IN;

    g_source_set_priority (source, CLUTTER_PRIORITY_EVENTS);
    g_source_add_poll (source, &event_poll_fd);
    g_source_set_can_recurse (source, TRUE);
    g_source_attach (source, NULL);
}

```

The `prepare()` and `check()` functions are a bit boring, but the `dispatch()` is worth a look:

```
static gboolean
event_dispatch (GSource *source, GSourceFunc callback, gpointer user_data)
{
    ClutterEvent *event;
    XEvent xevent;

    clutter_threads_enter ();

    while (!clutter_events_pending () && XPending (dpy)) {
        XNextEvent (dpy, &xevent);

        /* here the trickiness */
        if (xevent.xany.window == input) {
            xevent.xany.window = stage_win;
        }

        clutter_x11_handle_event (&xevent);
    }

    if ((event = clutter_event_get ())) {
        clutter_do_event (event);
        clutter_event_free (event);
    }

    clutter_threads_leave ();

    return TRUE;
}

```

We just pick off events, translating the input to the stage window, then give the events to clutter.

Obviously this is a crapload of code just to shunt events around; normally with clutter this is not necessary, but with the X compositing architecture being like it is, we have to roll our own GSource to do the event translation.

This brings me back to the beginning, `prep_clutter`:

```
GType texture_pixmap_type;
void prep_clutter (int *argc, char ***argv)
{
    clutter_x11_disable_event_retrieval ();
    clutter_init (argc, argv);
    clutter_x11_add_filter (event_filter, NULL);

    if (getenv ("NO_TFP"))
        texture_pixmap_type = CLUTTER_X11_TYPE_TEXTURE_PIXMAP;
    else
        texture_pixmap_type = CLUTTER_GLX_TYPE_TEXTURE_PIXMAP;
}

```

Here we see that you have to turn off event retrieval before calling `clutter_init`. Then, we tell Clutter to call a `event_filter` whenever it gets an X event. I'll get back to that function in a minute.

I suppose that this is as good a time as any to speak of getting window contents into OpenGL. The basic idea is that since we're already storing all of the window's pixels into a backing pixmap (because we are redirecting all output), we don't actually need to transfer the pixels back over the wire to the compositor to have the compositor show the window on the screen. There is a GLX extension, [texture from pixmap](http://www.opengl.org/registry/specs/EXT/texture_from_pixmap.txt) or TFP, which tells the libGL implementation to get a texture's contents from a pixmap that it already knows about, automatically updating when the pixmap updates.

I digress for a moment to mention something that I did not know when I was getting into all of this. There are two kinds of GLX rendering, direct and indirect. Perhaps you recall these words from running `glxinfo` to see whether your GL implementation has hardware acceleration or not. In fact, direct vs. indirect exists on a different axis as accelerated vs. software rendering. What "indirect rendering" means is that instead of talking directly to the graphics card through the kernel, a GL application is sending all of its commands over the wire to the X server, which then does the rendering.

But that server-side rendering may be accelerated; indeed, that was the whole point of [AIGLX](http://en.wikipedia.org/wiki/AIGLX). Technically, with free drivers, this is implemented via having the server's GLX rendering use the same DRI library as libGL does when doing direct rendering. It is the DRI library which does the accelerated communication with the hardware-specific kernel module (the DRM module).

Indirect rendering is slower, however, and it is advantageous to use direct rendering when possible. The problem comes when wanting to use TFP and direct rendering; if I want to bind a GL texture to a pixmap corresponding to some other application, and I am not going to go through the X server to do so, then obviously the *kernel itself* has to know about X drawables. If I have a named pixmap open from one process that was created from another process, there needs to be a unified memory manager in the kernel:

> It's a crude approximation but the most crucial difference between the nvidia architecture and DRI/DRM is that nvidia actually have a memory manager - and a unified one at that. Without a memory manager it's impossible to allocate offscreen buffers (hence, no pbuffers or fbos) and without a unified memory manager it's impossible to reconcile 2D and 3D operations (hence no redirected Direct Rendering). The Accelerated Indirect GLX feature that the freetards were busy raving about is an endless source of confusion - and ultimately a hack to workaround their lack of a memory manager.

(That from [Linux hater](http://linuxhaters.blogspot.com/2008/06/nitty-gritty-shit-on-open-source.html), someone I have warmed to in recent weeks -- I basically agree with [Jeremy Allison's position](http://blogs.zdnet.com/BTL/?p=9370).)

Anyway. You can't do direct TFP with free drivers, is the conclusion of that digression. That's OK, because you can always force indirect rendering via setting `LIBGL_ALWAYS_INDIRECT=1` in your environment. But you can't do [indirect rendering in Xephyr](https://bugs.freedesktop.org/show_bug.cgi?id=16559), only direct, so it's tough to test out window managers. Fortunately, you can get window contents into clutter without TFP, using the fallbacks -- hence the `NO_TFP` check in `prep_clutter`.

**ánimo, peregrino**

OK, at this point we're almost there. Here's the event handler:

```
static ClutterX11FilterReturn
event_filter (XEvent *ev, ClutterEvent *cev, gpointer unused)
{
    switch (ev->type) {
    case CreateNotify:
        window_created (ev->xcreatewindow.window);
        return CLUTTER_X11_FILTER_REMOVE;

    default:
        return CLUTTER_X11_FILTER_CONTINUE;
    }
}

```

It's pretty simple, an X event is a tagged union. We respond to the CreateNotify events, which come because we selected for SubstructureNotifyMask on the root window. It turns out we don't need any more events, because clutter handles the rest:

```
static void
window_created (Window w)
{
    XWindowAttributes attr;
    ClutterActor *tex;

    if (w == overlay)
        return;

    XGetWindowAttributes (dpy, w, &attr);
    if (attr.class == InputOnly)
        return;

    tex = g_object_new (texture_pixmap_type, "window", w,
                        "automatic-updates", TRUE, NULL);

    g_signal_connect (tex, "notify::mapped",
                      G_CALLBACK (window_mapped_changed), NULL);
    g_signal_connect (tex, "notify::window-x",
                      G_CALLBACK (window_position_changed), NULL);
    g_signal_connect (tex, "notify::window-y",
                      G_CALLBACK (window_position_changed), NULL);

    {
        gint mapped, destroyed;
        g_object_get (tex, "mapped", &mapped, NULL);
        if (mapped)
            window_mapped_changed (tex, NULL, NULL);
    }
}

```

Once we create the window, Clutter will listen for changes in its state -- resizes, maps or unmaps, and ultimately its destruction. (Or at least it will, once [#1020](http://bugzilla.o-hand.com/show_bug.cgi?id=1020) is applied.) Here we connect with the minimum bits necessary to make a window usable. Finally, the functions to show, hide, and resize windows:

```
static void
window_position_changed (ClutterActor *tex, GParamSpec *pspec, gpointer unused)
{
    gint x, y, window_x, window_y;
    g_object_get (tex, "x", &x, "y", &y, "window-x", &window_x,
                  "window-y", &window_y, NULL);
    if (x != window_x || y != window_y)
        clutter_actor_set_position (tex, window_x, window_y);
}

static void
window_mapped_changed (ClutterActor *tex, GParamSpec *pspec, gpointer unused)
{
    gint mapped;
    g_object_get (tex, "mapped", &mapped, NULL);

    if (mapped){
        clutter_container_add_actor (CLUTTER_CONTAINER (stage), tex);
        clutter_actor_show (tex);
        window_position_changed (tex, NULL, NULL);
    } else {
        clutter_container_remove_actor (CLUTTER_CONTAINER (stage), tex);
    }
}

```

**final notes**

Code is [here](//wingolog.org/pub/mini-clutter-wm.c), compile as:

```
 gcc `pkg-config --cflags --libs clutter-glx-0.8` \
     -o mini-clutter-wm mini-clutter-wm.c

```

You might want to run it in a Xephyr; do it with the script [here](//wingolog.org/pub/run-xephyr):

```
 ./run-xephyr ./mini-clutter-wm

```

Joey again:

> So ideally, "I switched to a new window manager" doesh't mean "my screen has some different widgets on it now". It means "I'm looking at the screen with new eyes."

This blog is already way too long, so I'll revisit interface concepts at some point in the future. For now I just wanted to pull together this knowledge in one place. Happy hacking!

## related articles

- [introducing griddy](/archives/2008/07/31/introducing-griddy)
- [compost, a leaf function compiler for guile](/archives/2014/02/18/compost-a-leaf-function-compiler-for-guile)
- [closedgl particle simulation in guile](/archives/2013/02/16/opengl-particle-simulation-in-guile)
- [guile ? clutter ? quoi ?](/archives/2008/07/11/guile-clutter-quoi-)
- [regarding decadence](/archives/2008/06/10/regarding-decadence)
- [allocate, memory (part one of n)](/archives/2008/04/10/allocate-memory-part-one-of-n)

### 15 responses

1. iain says:[26 July 2008 11:20 PM](#0aa2d8d8ac63d54cd6f00432250267fd356a0c25)

   feel free to join in the fun with the metacity compositor. Both in making a design thats useful for XRender and 3D backends :)

   If you want.

2. [Kevin Lange](http://oasis-games.com) says:[27 July 2008 0:45 AM](#2d9553213ba98c5f16e6b426832c2f7b1f3f1428)

   Can't redirect input? I beg to differ. http://youtube.com/watch?v=BrK4c7iFJLs

   Merged? Maybe not. Existant? Yes. Working? Yes.

   Can't do textureFromPixmap with free drivers? Ha! http://youtube.com/watch?v=4SoyuG4ygmI

   I rest my case.

3. [daniels](http://www.fooishbar.org) says:[27 July 2008 3:08 AM](#41e57b4fe68e70c14dce3ad39e7ec107a68897ae)

   In fairness, the Intel driver now does zero-copy TFP with DRI2.

4. [wingo](http://wingolog.org/) says:[27 July 2008 11:08 AM](#7984e0fd6fb2a1043576d530a61e25014119c602)

   @kevin: I do use TFP with free drivers. But you have to use indirect rendering, which you can't do with Xephyr. Not sure what your point is, I don't think we disagree.

   @daniels: Neat. I think I have that, but since I'm not dogfooding yet and just running things through Xephyr, I haven't been able to experience that goodness yet :)

5. [mallum](http://butterfeet.org) says:[27 July 2008 1:14 PM](#23f502e5ec28d0f588398e1a0a97704424f89edd)

   Really nice post sir.

6. [Kevin Lange](http://oasis-games.com) says:[27 July 2008 5:40 PM](#567c92dc823baa21282f92cf0a06736e58b91bc8)

   @wingo: Sorry, missed two important words in my response: That's /direct rendered/ TFP under DRI2 in my video, same as what Daniel pointed out.

   But you're not missing out on much. It crashes quite often, temporarily bricking your hardware until a reboot and hasn't been updated in months.

7. [behdad](http://behdad.org) says:[27 July 2008 7:31 PM](#db34d92ae73b5493fc323965832a8b674c788804)

   Very nice and informative post. Thanks sir.

8. Robert Bragg says:[29 July 2008 6:09 PM](#4944eeaea85557cff68559714898e7d077ae58f2)

   Excellent post!

   Theoretically I think if you remove your call to clutter\_x11\_disable\_event\_retrieval, then I think you can remove the code that sets up a new GSource (so attach\_event\_source and event\_dispatch).

   What clutter\_x11\_disable\_event\_retrieval does is remove clutters own GSource, and disable the call to XSelectInput on the stage. You don't need to worry about the latter, since you are making your overlay window into an output only window anyway, so I think you are currently left manually replacing the GSource you are disabling.

   I'd be interested to hear if that works for you.

   kind regards

9. [wingo](http://wingolog.org/) says:[29 July 2008 6:23 PM](#b3d833f3dfed8ee6919b2b115dfb7519d94c7e94)

   Hey Robert! There is the slight wrinkle of the event translation between the input and stage windows, but I can probably take care of that in the event filter, now that I think about it. I'll give it a try. I'm all about pushing the maintainance burden off to smarter people than I ;-)

10. [nornagon](http://nornagon.net) says:[26 August 2008 12:36 PM](#227d2f41e1f828ef17ef58ad20fe2396dc7b5e3c)

    I can't get this to work under Xephyr (haven't tried it under a 'real' X server, though). I compiled as you suggested, and ran thus:

    NO\_TFP=1 ./run-xephyr ./mini-clutter-wm

    When I started up an xterm (with DISPLAY=:3 xterm), I get the following errors:

    (./mini-clutter-wm:16594): ClutterX11-WARNING \*\*: Unable to query pixmap: 400007

    (./mini-clutter-wm:16594): GLib-GObject-WARNING \*\*: IA\_\_g\_object\_get\_valist: object class \`ClutterX11TexturePixmap' has no property named \`mapped'

    I'm using Debian sid, clutter 0.8.0, Xephyr 1.4.2 and binary nvidia drivers.

11. [wingo](http://wingolog.org/) says:[26 August 2008 7:02 PM](#1c52c78cf29fef1fbe6f4ed09e2e2812e3ce6e1f)

    nornagon: It seems they still haven't applied the patch from [bug #1020](http://bugzilla.openedhand.com/show_bug.cgi?id=1020) yet. I'll re-ping; it's probably just oversight.

12. [nornagon](http://nornagon.net) says:[26 August 2008 10:55 PM](#aa5c33453c8f85350395b5575e31085ee5caf0eb)

    I applied that patch to svn clutter and tried again. Windows show up now, but they don't receive any keyboard input. Also, xterm doesn't work properly:

    (./mini-clutter-wm:9202): ClutterX11-WARNING \*\*: Failed to get XImage of pixmap: 400008, removing

    and running uxterm crashes Xephyr:

    X Error of failed request: BadWindow (invalid Window parameter)

    Major opcode of failed request: 3 (X\_GetWindowAttributes)

    Resource id in failed request: 0x600004

    Serial number of failed request: 80

    Current serial number in output stream: 81

    Locking assertion failure. Backtrace:

    #0 /usr/lib/libxcb-xlib.so.0 \[0xb77db767\]

    #1 /usr/lib/libxcb-xlib.so.0(xcb\_xlib\_lock+0x2e) \[0xb77db81e\]

    #2 /usr/lib/libX11.so.6 \[0xb7a77dc9\]

    #3 /usr/lib/libX11.so.6(XESetCloseDisplay+0x44) \[0xb7a59e14\]

    #4 /usr/lib/libGL.so.1 \[0xb79c97c9\]

13. Rajeev Ranjan says:[22 October 2009 2:47 AM](#bd7c691df09cc6095d549d1fc7778193fd5f5353)

    Hi,

    I am interested in knowing the basic of compositor and I am completely new to this area although I am little familer with X11. What I mean by this is that I want to know the steps involved in compositing in order.

    In fact I am planning to create a compositor with effects for window transition based on events(map,unmap,configure etc) and full screen based effects like water in compiz on complete screen based on pointer event. I want to render the effect using GLES. Please help me in this regard by either giving these details or by providing a reference pointer.

    Thanks.

    Regards,

    Rajeev

14. [Paolo](http://www.eurika.net) says:[19 June 2011 4:10 PM](#45a8f221e8172b12d761dbed8e435f6daa9de138)

    Hi Wingo, i'm working on a simple compositor starting from your code, but i have 2 problems.

    \- if i add a clutter actor to stage before showing stage, all is ok, but if i create any actor after stage has been showed the stage is moved down for quite 500 pixels (and i can't put nothing i upper part of the screen)

    \- even with X11 and GLX textures, if i start a window that used opengl (for example glxgears) all stop.

    Can you give me some ideas for the direction to search solutions?

    (i'm working with clutter 1.0 now)

    Regards,

    Paolo

15. [Ivan Vučica](http://ivan.vucica.net/) says:[25 December 2011 9:19 PM](#65565fe902351eaba064236b3fb9a183e9b81f3c)

    Hello Wingo,

    I'm currently playing with Ubuntu 10.04's stock Clutter (apparently 1.0). I've fixed several problems with renamed functions between Clutter 0.8 and 1.0. However, there appears to be a big issue: it doesn't seem any "notify::mapped" signals are sent, and window\_mapped\_changed() does not get called.

    Are you familiar with any significant changes to Clutter which might prefer this signal from being sent? Could you post any info you have, possibly pinging me via mail?

    By the way, it looks like all the way into 2011, this article is the best and simplest source of information on writing a simple compositing window manager :)

Comments are closed.
