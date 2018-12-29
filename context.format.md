# Format

## Printer.java

```java
package org.springframework.format;

import java.util.Locale;
/**
 * 打印器
 */
public interface Printer<T> {

	String print(T object, Locale locale);

}

```

## Parser.java

```java
package org.springframework.format;

import java.text.ParseException;
import java.util.Locale;
/**
 * 解析器
 */
public interface Parser<T> {

	T parse(String text, Locale locale) throws ParseException;

}

```

## Formatter.java

```java
package org.springframework.format;
/**
 * 格式化器
 */
public interface Formatter<T> extends Printer<T>, Parser<T> {

}

```

