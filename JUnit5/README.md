# JUnit 5 - Jupiter

This is to document the ways I personally have used JUnit 5 in ARCADE, along with some tips for Java itself which can be helpful.

## Maven dependencies

I use three dependencies for JUnit 5:

```
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-engine</artifactId>
  <version>5.8.0-M1</version>
  <scope>test</scope>
</dependency>
```

This is the JUnit 5 core engine. This dependency is sufficient for basic tests.

```
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-params</artifactId>
  <version>5.8.0-M1</version>
  <scope>test</scope>
</dependency>
```

This is the JUnit 5 parameterization extension. It is used for writing parameterized tests.

```
<dependency>
  <groupId>com.ginsberg</groupId>
  <artifactId>junit5-system-exit</artifactId>
  <version>1.0.0</version>
  <scope>test</scope>
</dependency>
```

Finally, this is a third-party extension that includes an utility to test for system exit.

## Basic Tests

A basic, unparameterized test will look something like this:

```
@Test
public void getPackageNameFromJavaFileTest1() {
  // File does not contain any package names
  char fs = File.separatorChar;
  String filePath = "." + fs + "src" + fs + "test" + fs + "resources" + fs + "FileUtilTest_resources" + fs + "textFile.txt";
  String result = assertDoesNotThrow(() -> { return FileUtil.getPackageNameFromJavaFile(filePath); });
  assertEquals(null, result);
}
```

First, you annotate a method as `@Test`. Then, preferably, you place a comment explaining what is the purpose of that test.

To check for exceptions, you may use the functional interfaces `assertDoesNotThrow` and `assertThrows`. `assertDoesNotThrow` checks that no exceptions are thrown by a statement: you only need to pass a simple lambda expression for which an exception might be thrown. `assertThrows` checks that a specific exception is thrown by a statement: you pass the exception type, followed by the lambda expression. For example:

```
assertThrows(IOException.class, () -> FileUtil.getPackageNameFromJavaFile(filePath));
```

To check the results against an oracle, you may use `assertEquals`: remember that the first parameter must always be the oracle, and the second be the result. If you invert the parameters, not only will my linter yell at me, the reports will also look inverted, which can make reading them difficult and confusing.

Finally, to check boolean expressions, you can use `assertTrue` and `assertFalse`, passing the expression in. An example of usage for this is when you want to test whether a `file.exists()`.

## Parameterized Tests

When several tests run practically the same way except for a few parameters, you can use the jupiter-params to condense them. A parameterized test looks something like this:

```
@ParameterizedTest
@CsvSource({
  // Simple filename test
  "fileName.suffix,fileName",
  // Filename without suffix
  "fileName,fileName",
  // Filename inside directory, relative path, Windows format
  "directory///fileName.suffix,fileName",
  // Filename inside directory, absolute path, Unix format
  "///dir1///dir2///dir3///fileName.suffix,fileName",
})
public void extractFilenamePrefixTest1(String test, String oracle) {
  // Test using the String overload
  test = test.replace("///", File.separator);
  oracle = oracle == null ? "" : oracle;
  String result = FileUtil.extractFilenamePrefix(test);
  assertEquals(oracle, result);
}
```

First, you declare the method to be a `@ParameterizedTest`, and then declare an `@ArgumentsSource` containing the values to be used as arguments. There are various guides on how to use these argument sources, such as https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-sources, but generally speaking, I prefer the `@CsvSource`. This is a format that passes a set of comma-separated values, all of which must be literals, and each of which is split and fed into the parameters of the `@ParameterizedTest`. For example, `fileName.suffix,fileName` will be parsed into two separate `String`s, `fileName.suffix` and `fileName`, which are then passed as the arguments `test` and `oracle`.

The test method itself looks mostly the same as a [basic test](#basic-tests), except for a slight hack: because the `@CsvSource` only accepts literals, I use wildcards which I can replace for variables inside the method itself. In the above example, `///` is used as a wildcard for a `File.separator`, and is replaced within the method's body. Another detail is that one cannot pass the empty `String` in a `@CsvSource`: it will instead pass `null` when nothing is found. If necessary, one may test for this in the body and replace the value, as shown above: `oracle = oracle == null ? "" : oracle;`.

## Checking for system exit

Sometimes, the expected behavior of a certain test is for the system to exit, typically because a critical error occurred. While in the vast majority of cases the best treatment to this is to throw a `RuntimeException` rather than a direct `System.exit`, it is still important to be able to test for this: an uncaught `System.exit` will actually exit the entire test VM, prematurely ending the tests.

```
@Test
@ExpectSystemExit
public void checkFileTest7() {
  // Input is not an existing file, create = false, exitOnNoExist = true
  String fs = File.separator;
  String path = "src" + fs + "test" + fs + "resources" + fs + "FileUtilTest_resources" + fs + "nonExist.txt";
  FileUtil.checkFile(path, false, true);
}
```

The ginsberg library provides the `@ExpectSystemExit` annotation, which observes the test's execution for calls of `System.exit` and wraps them in a way similar to JUnit assertions. If a test is annotated to `@ExpectSystemExit`, then it will pass if the execution results in `System.exit` and fail otherwise. Other assertions can still be included in such a test, and will take priority over `System.exit` if they occur first. If multiple `System.exit` codes can happen during the test's execution and only one of them is desirable, one may also use the `@ExpectSystemExitWithStatus(#)` annotation.

See https://todd.ginsberg.com/post/testing-system-exit/ for more information.

## Note on the use of the `Comparable` interface

The `Comparable` Java interface is typically used for sorting. While it can be used for simple comparisons, it is typically best to simply override the `equals` method.

```
@Override
public int compareTo(RsfCompare rsf1) {
  // Returns 0 if contents of the 2 rsf files are the same (regardless of order)
  if (this.rsfSet.equals(rsf1.rsfSet))
    return 0;
  else
    return 1;
}
```

The above is an example of using the `Comparable` interface to check for equality. The same result can be achieved by the following overload of `equals`:

```
@Override
public boolean equals(Object o) {
  // Check if the given object is of the correct type
  if (!(o instanceof RsfCompare)) return false;

  RsfCompare toCompare = (RsfCompare)o;
  return this.rsfSet.equals(toCompare.rsfSet);
}
```

To my knowledge, there is no practical advantage to either approach. This is just for information's sake, as a large part of the Java community considers the `Comparable` interface to be obsolete.