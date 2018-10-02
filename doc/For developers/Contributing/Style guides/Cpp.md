We aim to use a subset of all C++ features. This includes most features present in C++ up to C++11.

# General

## Platform-specific code

* Use #ifdef to differentiate Windows and non-Windows code
* Try to implement Windows-specific code on the lowest possible abstraction layer so that the higher ones can be platform agnostic.
* When providing alternate body of a function, use one function header/signature but two implementations in its body, encased in #ifdef guards.

## Practices

* Use class declarations in header files if the definition is not necessary.
```
class BgpAttr;
```
* In header files, use #ifdef guards to prevent any issues arising from including the same header twice
```
#ifndef ctrlplane_ksync_sock_h
#define ctrlplane_ksync_sock_h
// Some code
#endif
Use of many for-each variants instead of iterating over an index when accessing an array
for(auto &element : array) {
  //do something...
}
auto itr = strvec.begin();
while(itr != strvec.end()) {
  //do something...
  ++itr;
}
```

# Style

## Files

* Use .cc files for C++ source code
* Use .h files for C++ header code

##Naming

### Files
* Files should be named in snake case.
```
iroute_aggregator.h
Classes
Classes should be named in camel case with the first letter capital
class IRouteAggregator {
};
```
### Functions/Methods
* Methods and functions should be named in upper camel case
```
class IRouteAggregator {
    void Initialize();
};
```
### Variables/Fields
* Variables and fields should be named according to the snake case convention
** Note: Private fields often have a single underscore appended, as this makes creating a getter/setter more intuitive
# Design
## Paradigm
* Agent is designed with traditional object-oriented model of C++ application, therefore we should try to maintain that style.
```
Agent does not use exceptions, instead relying on old-school return code error handling.
int func()
{
    if (error)
        return -ENOMEM;
}
```
* Agent does use virtual and abstract (pure virtual) classes.
```
class Class {
public:
virtual void Func() = 0;
virtual void Func2();
void Func3();
private:
};
```
* Agent uses proper visibility settings, ie. uses public, private and protected specifiers in class definitions.
* Templating is used and encouraged.
