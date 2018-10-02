We try to maintain the same coding style as the rest of vRouter wherever we can, but still use conventions used by Windows Driver Kit wherever applicable (regarding variable names, P convention instead of * for specifying pointers to native Windows structures).

Below every rule, there is an example illustrating it with a code sample.

# General

## Styling

* Use spaces, not tabs.
* One indentation level is 4 spaces
* Every line should end with a newline character ('\n')
* Newlines commited into the remote repository should be Unix-like (newline character), avoid Windows-like newlines (carriage return; newline)
* Avoid magic numbers, unless there is a very good reason.
* Use define directives instead.
* Statements with blocks (if, while, for, do...) should have the opening bracket in the same line
* Statements with only one line instead of a whole block can skip brackets if it improves readability
* Statements if/else should not skip brackets unless it can be done for both if, else and all eventual else ifs
```
if (condition)
	do_something();
else
	do_nothing();


if (condition) {
	do_something();
} else {
	i+= 1;
	do_nothing();
}


do {
	ASSERT(FALSE);
	nobody_uses_this_feature();
} while(condition);
```
* If, for, while and similar statements should not be too long. This means, for example creating variables for parts of statement being checked
```
BOOLEAN usable = (value >= MIN_VALUE && value <= MAX_VALUE);
BOOLEAN safe = (state == STABLE);
if (usable && safe)
	do_something();
```

## Files

* Files in 'windows' directory are free to modify and do not have to be platform portable (they have to be Windows-compatible)
* Other files need to be platform independant. This may be done in #ifdef manner, but more elegant solutions are preferable
* For example callbacks
* Function names should be:
vr_filename.c or vr_filename.h
** Please note vrouter_mod.c is an exception to this
* Files should be in directory:
** windows for Windows source files (.c extension)
** include for header files (.h extension)
** dp-core for multiplatform source files (we don't forsee creating any)
*For header files, use guards to make sure there is no problem if they are included two or more times.
```
#ifndef __VR_DEFS_H__
#define __VR_DEFS_H__
// Some code
#endif
```
* Use #ifdef, not #pragma once for guards

## Naming

### Functions
* Name functions in snake case, names starting with vr_ or win_. Please note functions in vrouter_mod.c ATM don't follow this rule because of SxLibrary requirements. This will be changed when we get rid of it.
vr_reinject_packet
* vr_ variant should be used for multiplatform functions
* win_ variant should be used for Windows-specific functions.
* When defining function, place the type in one line and the rest of it in another line.

```
unsigned int
vr_reinject_packet(struct vr_packet *pkt, struct vr_forwarding_md *fmd)
```

* When declaring functions, use the extern keyword and write everything in one line.
```
extern unsigned int vr_some_function();
```
* It is preferential but not required to declare which parameters of a function are constant.
```
int
vr_function(const char* address)
```
### Variables
* We avoid constant literal variables. Instead, one should avoid magic numbers using #define directive. Const structs are OK.
```
#define MAX_SIZE 32
```
* Global variables can be used whenever necessary, especially when the function being implemented will be called by some pre-existing dp-core code without supplying required arguments. However, it is preferential to avoid them. Static variables are lesser evil. Please check if a global variable can be replaced with a static one
* Variables should be named using snake case.
```
LARGE_INTEGER current_gmt_time;
```
* It is preferential but not required to declare constant variables as such
```
const unsigned int num_of_tries = input + 5;
```
* If a value is known to be always positive, one should use unsigned variable. Same for bitfields.
```
unsigned int i = 5;
```
* Variables should be initialized before reading them.

## Types
* Existing types should be used as-is
```
PNET_BUFFER_LIST nbl;
```
* New types and types declared by us should use snake case and begin with vr_.
```
struct vr_struct {};
```
* Do not typedef function, use struct as a part of the name
```
struct vr_router* router = vrouter_get(0);
```
### Comments
* Both block and line comments are allowed
```
int i = pointer->field; // Pointer cannot be null

/* TODO: Fix this */
```
* Non-obvious logic should be commented
* Do not leave commented out code unless there is a very good reason.
* Platform-specific code
* Use #ifdefs to specify code that should be compiled on only selected platforms
```
#ifdef __KERNEL__
```
* Prefer #ifdef to #if defined unless more specific statement checking if required.
```
#if defined(_WINDOWS) || defined(__FreeBSD__)
```
* Do not use compiler or linker specific features in code if they would break compatibility with other tools. For example, most #pragma statements.
```
#ifdef _WINDOWS
#pragma warning(disable : 4018) // Disable warning about signed/unsigned mismatch
#endif
```
* Try not to modify existing dp-core code if possible, otherwise modify it:
* Preferential: Creating additional callbacks (can be NULL or empty functions on Linux/FreeBSD)

### Using #ifdefs

* Try not to use os, compiler or linker specific features if possible
* If there is already some code implemented using them, consider reimplementing a native Linux function using Windows libraries. This function should only be compiled on Windows as Linux-like systems will provide their own implementation. Either of these two options will work
* Write it in a file that will only be included on Windows
```
#ifdef the implementation
If the body of a function will be completely different on Linux and Windows, write one function header and footer and #ifdef the rest of the body
void print(const char* a)
{
#ifdef _WINDOWS
DbgPrint(a);
#else
printf(a);
#endif
}
```

* Windows-specific code should be properely ASSERTed. Use this function or ASSERTMSG to check your assumptions

## Other points

* If a possible data-race condition arises, find the smallest critical section and wrap it into a mutex, semaphore or a RW lock.
* Functions should return:
** In Windows-specific code: NDIS_STATUS and all effects via pointers unless the function cannot fail
** In multiplatform code: regular return value if functions succeeds and -errno if the function fails