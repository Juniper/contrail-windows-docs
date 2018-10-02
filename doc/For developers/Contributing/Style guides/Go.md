Since docker driver written in Go is the only module of OpenContrail we've written completely by ourselves, we can introduce the best programming practices here, without regard for compatibility with legacy code.

For issues not discussed here, look at Official Code Review Comments.

# General

## Coding style

* Avoid nesting if possible
* Try to exit from the function early in a case of an error
* Use tabs for indentations.
* One tab is one indentation level.
* Code should be as compliant with gometalinter.
* If a line would be longer than 100 characters (cound tab as 4 spaces for this), divide it into a few lines, so every line is shorter than said length.

## Errors

* Handle errors as soon as possible
* Functions that are not sure to be successful should return error as one of returned values

```
func SomeFunc(param string) (string, error) {
```

## Other guidelines

* Try to use libraries to do required functionality, instead of calling PowerShell commands or otherwise using external features if possible
* Often it's not possible
* Use log module to provide logging capability
* Remember to use different message levels correctly, ie. error for errors, info for regular info, debug for info which should usually be hidden, etc.

```
log.Infoln("Deleting HNS network", hnsID)
```

## Design

### Modules

* Keep modules independent if possible
* Otherwise, make the dependency explicit
* Dependencies can often be avoided by using interfaces
* When importing not inbuilt modules (eg. from github), keep a copy of them in vendor directory. Use govendor to upkeep it.

### Functions

* Functions should only accept as arguments what they really need, ie. io.Writer instead of *os.File
* Dependencies can be avoided by using interfaces
* Avoid functions concurrent by default. By exposing synchronous API one can easily choose how to call the function, the other way around doesn't work.
* Use chans to communicate with goroutines
* Unexported functions should be named in camel case with first letter lowercase
* Functions should be named in camel case with first letter capital for exported functions

```
func CreateHNSNetwork(configuration *hcsshim.HNSNetwork) (string, error) {
```

## Testing

* All features should be tested if possible
* Features implemented in file.go should be tested in file_test.go
* Used frameworks should be ginkgo and gomega
