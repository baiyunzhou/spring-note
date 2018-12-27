# Demo

```java
package com.zby;

import javax.xml.bind.annotation.XmlRootElement;

import org.springframework.beans.annotation.AnnotationBeanUtils;

@XmlRootElement(name = "zby", namespace = "com.zby")
public class AnnotationBeanUtilMain {

	public static void main(String[] args) {
		XmlRootElement xmlRootElement = AnnotationBeanUtilMain.class.getDeclaredAnnotation(XmlRootElement.class);
		AnnoBean annoBean = new AnnoBean();
		AnnotationBeanUtils.copyPropertiesToBean(xmlRootElement, annoBean);
		System.out.println(annoBean);

	}

	static class AnnoBean {
		private String name;
		private String namespace;

		public String getName() {
			return name;
		}

		public void setName(String name) {
			this.name = name;
		}

		public String getNamespace() {
			return namespace;
		}

		public void setNamespace(String namespace) {
			this.namespace = namespace;
		}

		@Override
		public String toString() {
			return "AnnoBean [name=" + name + ", namespace=" + namespace + "]";
		}

	}
}

```

```console
AnnoBean [name=zby, namespace=com.zby]
```

