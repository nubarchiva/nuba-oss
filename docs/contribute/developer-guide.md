# 🧑‍💻 Developer Guide

This guide helps you get started developing nubarchiva.

## Dependencies

Before you start contributing to nubarchiva, make sure you have the following tools and programming
languages installed.

Required tools:

- `git`

Required programming languages:

- Java 11
- Maven 3.5.0+
 
## How to run in your local environment

Run the following command to build this project:

```bash
mvn clean verify
```

Pass the `-Dquick` option to skip all non-essential plug-ins and create the output artifact as
quickly as possible:

```bash
mvn clean verify -Dquick
```

Run the following command to format the source code and organize the imports as per the project's
conventions:

```bash
mvn process-sources
```

Run the following command to add license header info:

```bash
mvn -Pcode-format generate-sources
```

### Git workflow basic

Here's a general nubarchiva git workflow:

1. Create an issue and describe features you want -> [nubarchiva issues](https://github.com/nubarchiva/nuba-oss/issues)
2. Fork this project -> [Fork nubarchiva project](https://github.com/nubarchiva/nuba-oss/fork)
3. Create your feature branch (`git checkout -b my-new-feature`)
4. Commit your changes (`git commit -am 'Add some features'`)
5. Publish the branch (`git push origin my-new-feature`)
6. Create a new Pull Request -> [Create pull request across forks](https://github.com/nubarchiva/nuba-oss/compare)
