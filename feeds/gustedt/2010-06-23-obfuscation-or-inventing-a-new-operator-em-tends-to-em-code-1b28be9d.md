---
title: Obfuscation or inventing a new operator <em>tends to</em> <code>operator&nbsp;--&gt;</code>
url: https://gustedt.wordpress.com/2010/06/23/obfuscation-or-inventing-a-new-operator-tends-to-operator/
published: "2010-06-23T09:21:02Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=93
---

# Obfuscation or inventing a new operator <em>tends to</em> <code>operator&nbsp;--&gt;</code>

I found a really nice one in this discussion [here](http://stackoverflow.com/questions/1642028/what-is-the-name-of-this-operator). Basically the idea is that you may format operators a bit to show this code, which is valid C99 and C++:

```
for (unsigned x = 10; x --> 0;)
     printf("%u\n", x);

```

Ain’t that cute?

Now I was thinking that in C++ we could really obfuscate that much better by inventing some helper class. I came up with the following class `Heron` that \`converges’, (written as `aHeron --> eps`) towards the square root of the initial value.

Enjoy.

```
#include <iostream>
#include <math.h>

using std::cout;
using std::endl;

struct Heron {
  double const a;
  double x;
  Heron(double _a) : a(_a), x(_a) { }
  operator double(void){ return x; }
};

class Heron_tmp;

inline
Heron_tmp operator--(Heron& h, int);

class Heron_tmp {
  friend class Heron;
  friend Heron_tmp operator--(Heron& h, int);
private:
  Heron* here;
  Heron_tmp(Heron& h) : here(&h) { }
public:
  inline int operator>(double err) const;
};

inline
Heron_tmp operator--(Heron& here, int) {
  return here;
}

inline
int Heron_tmp::operator>(double err) const {
  double& x = here->x;
  double const& a = here->a;
  x = (x + a/x) * 0.5;
  return fabs((x*x - a)/a) > err;
}

int main(void) {
  Heron aHeron(2.0);
  while (aHeron --> 1E-15)
    cout << (double)aHeron << endl;
}

```
