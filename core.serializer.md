# Serializer

## Serializer

```java
package org.springframework.core.serializer;

import java.io.IOException;
import java.io.OutputStream;

public interface Serializer<T> {

	void serialize(T object, OutputStream outputStream) throws IOException;

}

```

### DefaultSerializer.java

```java
package org.springframework.core.serializer;

import java.io.IOException;
import java.io.ObjectOutputStream;
import java.io.OutputStream;
import java.io.Serializable;

public class DefaultSerializer implements Serializer<Object> {

	@Override
	public void serialize(Object object, OutputStream outputStream) throws IOException {
		if (!(object instanceof Serializable)) {
			throw new IllegalArgumentException(getClass().getSimpleName() + " requires a Serializable payload " +
					"but received an object of type [" + object.getClass().getName() + "]");
		}
		ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
		objectOutputStream.writeObject(object);
		objectOutputStream.flush();
	}

}
```

## Deserializer

```java
package org.springframework.core.serializer;

import java.io.IOException;
import java.io.InputStream;

public interface Deserializer<T> {

	T deserialize(InputStream inputStream) throws IOException;

}

```

### DefaultSerializer.java

```java
package org.springframework.core.serializer;

import java.io.IOException;
import java.io.ObjectOutputStream;
import java.io.OutputStream;
import java.io.Serializable;

public class DefaultSerializer implements Serializer<Object> {

	@Override
	public void serialize(Object object, OutputStream outputStream) throws IOException {
		if (!(object instanceof Serializable)) {
			throw new IllegalArgumentException(getClass().getSimpleName() + " requires a Serializable payload " +
					"but received an object of type [" + object.getClass().getName() + "]");
		}
		ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
		objectOutputStream.writeObject(object);
		objectOutputStream.flush();
	}

}

```

## SerializingConverter.java

```java
package org.springframework.core.serializer.support;

import java.io.ByteArrayOutputStream;

import org.springframework.core.convert.converter.Converter;
import org.springframework.core.serializer.DefaultSerializer;
import org.springframework.core.serializer.Serializer;
import org.springframework.util.Assert;

/**
 * 序列化转换器，可以直接注册到转换服务上拿来用
 */
public class SerializingConverter implements Converter<Object, byte[]> {

	private final Serializer<Object> serializer;

	public SerializingConverter() {
		this.serializer = new DefaultSerializer();
	}

	public SerializingConverter(Serializer<Object> serializer) {
		Assert.notNull(serializer, "Serializer must not be null");
		this.serializer = serializer;
	}

	@Override
	public byte[] convert(Object source) {
		ByteArrayOutputStream byteStream = new ByteArrayOutputStream(1024);
		try  {
			this.serializer.serialize(source, byteStream);
			return byteStream.toByteArray();
		}
		catch (Throwable ex) {
			throw new SerializationFailedException("Failed to serialize object using " +
					this.serializer.getClass().getSimpleName(), ex);
		}
	}

}

```

## DeserializingConverter.java

```java
package org.springframework.core.serializer.support;

import java.io.ByteArrayInputStream;

import org.springframework.core.convert.converter.Converter;
import org.springframework.core.serializer.DefaultDeserializer;
import org.springframework.core.serializer.Deserializer;
import org.springframework.util.Assert;

/**
 * 反序列化转换器，可以直接注册到转换服务上拿来用
 */
public class DeserializingConverter implements Converter<byte[], Object> {

	private final Deserializer<Object> deserializer;

	public DeserializingConverter() {
		this.deserializer = new DefaultDeserializer();
	}

	public DeserializingConverter(ClassLoader classLoader) {
		this.deserializer = new DefaultDeserializer(classLoader);
	}

	public DeserializingConverter(Deserializer<Object> deserializer) {
		Assert.notNull(deserializer, "Deserializer must not be null");
		this.deserializer = deserializer;
	}

	@Override
	public Object convert(byte[] source) {
		ByteArrayInputStream byteStream = new ByteArrayInputStream(source);
		try {
			return this.deserializer.deserialize(byteStream);
		}
		catch (Throwable ex) {
			throw new SerializationFailedException("Failed to deserialize payload. " +
					"Is the byte array a result of corresponding serialization for " +
					this.deserializer.getClass().getSimpleName() + "?", ex);
		}
	}

}

```

## SerializationDelegate.java

```java
package org.springframework.core.serializer.support;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

import org.springframework.core.serializer.DefaultDeserializer;
import org.springframework.core.serializer.DefaultSerializer;
import org.springframework.core.serializer.Deserializer;
import org.springframework.core.serializer.Serializer;
import org.springframework.util.Assert;

/**
 * 序列化和反序列化委托
 */
public class SerializationDelegate implements Serializer<Object>, Deserializer<Object> {

	private final Serializer<Object> serializer;

	private final Deserializer<Object> deserializer;

	public SerializationDelegate(ClassLoader classLoader) {
		this.serializer = new DefaultSerializer();
		this.deserializer = new DefaultDeserializer(classLoader);
	}
	public SerializationDelegate(Serializer<Object> serializer, Deserializer<Object> deserializer) {
		Assert.notNull(serializer, "Serializer must not be null");
		Assert.notNull(deserializer, "Deserializer must not be null");
		this.serializer = serializer;
		this.deserializer = deserializer;
	}


	@Override
	public void serialize(Object object, OutputStream outputStream) throws IOException {
		this.serializer.serialize(object, outputStream);
	}

	@Override
	public Object deserialize(InputStream inputStream) throws IOException {
		return this.deserializer.deserialize(inputStream);
	}

}

```

## SerializationFailedException.java

```java
package org.springframework.core.serializer.support;

import org.springframework.core.NestedRuntimeException;

@SuppressWarnings("serial")
public class SerializationFailedException extends NestedRuntimeException {

	public SerializationFailedException(String message) {
		super(message);
	}

	public SerializationFailedException(String message, Throwable cause) {
		super(message, cause);
	}

}

```

