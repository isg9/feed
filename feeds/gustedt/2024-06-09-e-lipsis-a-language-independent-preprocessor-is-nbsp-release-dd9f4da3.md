---
title: 'EĿlipsis: a language independent preprocessor is&nbsp;released'
url: https://gustedt.wordpress.com/2024/06/09/ellipsis-a-language-independent-preprocessor-is-released/
published: "2024-06-09T15:42:28Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=4135
---

# EĿlipsis: a language independent preprocessor is&nbsp;released

[EĿlipsis](https://gitlab.inria.fr/gustedt/ellipsis) is an extension and continuation of the C preprocessor with the

aim to be useful for other technical languages, be they programming

languages or text processing languages.

There were several goals when developing eĿlipsis:

- Provide a complete specification of a preprocessor that is, as a specification, independent from the C or C++ specifications. This will be provided in a separate document, for which this code here serves as a reference implementation.
- Properly extend the preprocessor for other languages such that it can take special syntactic properties of these languages into account. Currently we have rudimentary support for lex and for general text processing languages, and more specifically for html and markdown.
- Develop a project that is fully written in C23 and uses the new features from there extensively.
- Extend the preprocessor to new features that make programming with it easier. Hopefully at some point it might be integrated into C and C++.
- Allow the use of modern Unicode to properly specify arithmetic formula and for technology and natural language names.
- Develop code that uses the new extensions to show their usefulness.
- Use the new features on the code of eĿlipsis itself (30k loc), to show that these extensions are functioning on a medium size project.

[https://gitlab.inria.fr/gustedt/ellipsis](https://gitlab.inria.fr/gustedt/ellipsis)the sources at INRIA’s gitlab server[https://gustedt.gitlabpages.inria.fr/ellipsis](https://gustedt.gitlabpages.inria.fr/ellipsis)documentation on the gitlab pages[https://gitlab.inria.fr/gustedt/ellipsis/-/boards](https://gitlab.inria.fr/gustedt/ellipsis/-/boards)issue trackers on different boardsBugsif things go wrongDocumentationif you lack an explanationFeature requestsif you want to do something or get something doneWhere to find eĿlipsis
