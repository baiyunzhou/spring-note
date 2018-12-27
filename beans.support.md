# Support

## Page

### SortDefinition.java

```java
package org.springframework.beans.support;

/**
 * 排序定义
 */
public interface SortDefinition {
	//排序属性名
	String getProperty();
	//是否忽略大小写
	boolean isIgnoreCase();
	//升序还是降序
	boolean isAscending();

}

```

#### MutableSortDefinition.java

```java
package org.springframework.beans.support;

import java.io.Serializable;

import org.springframework.util.StringUtils;

/**
 * 易变的排序定义。添加了升序复位支持。
 */
@SuppressWarnings("serial")
public class MutableSortDefinition implements SortDefinition, Serializable {

	private String property = "";

	private boolean ignoreCase = true;

	private boolean ascending = true;

	private boolean toggleAscendingOnProperty = false;

	public MutableSortDefinition() {
	}

	public MutableSortDefinition(SortDefinition source) {
		this.property = source.getProperty();
		this.ignoreCase = source.isIgnoreCase();
		this.ascending = source.isAscending();
	}

	public MutableSortDefinition(String property, boolean ignoreCase, boolean ascending) {
		this.property = property;
		this.ignoreCase = ignoreCase;
		this.ascending = ascending;
	}

	public MutableSortDefinition(boolean toggleAscendingOnSameProperty) {
		this.toggleAscendingOnProperty = toggleAscendingOnSameProperty;
	}

	public void setProperty(String property) {
		if (!StringUtils.hasLength(property)) {
			this.property = "";
		}
		else {
			// Implicit toggling of ascending?
			if (isToggleAscendingOnProperty()) {
				this.ascending = (!property.equals(this.property) || !this.ascending);
			}
			this.property = property;
		}
	}

	@Override
	public String getProperty() {
		return this.property;
	}

	public void setIgnoreCase(boolean ignoreCase) {
		this.ignoreCase = ignoreCase;
	}

	@Override
	public boolean isIgnoreCase() {
		return this.ignoreCase;
	}

	public void setAscending(boolean ascending) {
		this.ascending = ascending;
	}

	@Override
	public boolean isAscending() {
		return this.ascending;
	}

	public void setToggleAscendingOnProperty(boolean toggleAscendingOnProperty) {
		this.toggleAscendingOnProperty = toggleAscendingOnProperty;
	}

	public boolean isToggleAscendingOnProperty() {
		return this.toggleAscendingOnProperty;
	}


	@Override
	public boolean equals(Object other) {
		if (this == other) {
			return true;
		}
		if (!(other instanceof SortDefinition)) {
			return false;
		}
		SortDefinition otherSd = (SortDefinition) other;
		return (getProperty().equals(otherSd.getProperty()) &&
				isAscending() == otherSd.isAscending() &&
				isIgnoreCase() == otherSd.isIgnoreCase());
	}

	@Override
	public int hashCode() {
		int hashCode = getProperty().hashCode();
		hashCode = 29 * hashCode + (isIgnoreCase() ? 1 : 0);
		hashCode = 29 * hashCode + (isAscending() ? 1 : 0);
		return hashCode;
	}

}

```

### PropertyComparator.java

```java
package org.springframework.beans.support;

import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import org.springframework.beans.BeanWrapperImpl;
import org.springframework.beans.BeansException;
import org.springframework.util.StringUtils;

/**
 * 属性排序器
 */
public class PropertyComparator<T> implements Comparator<T> {

	protected final Log logger = LogFactory.getLog(getClass());

	private final SortDefinition sortDefinition;

	private final BeanWrapperImpl beanWrapper = new BeanWrapperImpl(false);

	public PropertyComparator(SortDefinition sortDefinition) {
		this.sortDefinition = sortDefinition;
	}

	public PropertyComparator(String property, boolean ignoreCase, boolean ascending) {
		this.sortDefinition = new MutableSortDefinition(property, ignoreCase, ascending);
	}

	public final SortDefinition getSortDefinition() {
		return this.sortDefinition;
	}


	@Override
	@SuppressWarnings("unchecked")
	public int compare(T o1, T o2) {
		Object v1 = getPropertyValue(o1);
		Object v2 = getPropertyValue(o2);
		if (this.sortDefinition.isIgnoreCase() && (v1 instanceof String) && (v2 instanceof String)) {
			v1 = ((String) v1).toLowerCase();
			v2 = ((String) v2).toLowerCase();
		}

		int result;

		// Put an object with null property at the end of the sort result.
		try {
			if (v1 != null) {
				result = (v2 != null ? ((Comparable<Object>) v1).compareTo(v2) : -1);
			}
			else {
				result = (v2 != null ? 1 : 0);
			}
		}
		catch (RuntimeException ex) {
			if (logger.isWarnEnabled()) {
				logger.warn("Could not sort objects [" + o1 + "] and [" + o2 + "]", ex);
			}
			return 0;
		}

		return (this.sortDefinition.isAscending() ? result : -result);
	}

	private Object getPropertyValue(Object obj) {
		// If a nested property cannot be read, simply return null
		// (similar to JSTL EL). If the property doesn't exist in the
		// first place, let the exception through.
		try {
			this.beanWrapper.setWrappedInstance(obj);
			return this.beanWrapper.getPropertyValue(this.sortDefinition.getProperty());
		}
		catch (BeansException ex) {
			logger.info("PropertyComparator could not access property - treating as null for sorting", ex);
			return null;
		}
	}
rows java.lang.IllegalArgumentException in case of a missing propertyName
	 */
	public static void sort(List<?> source, SortDefinition sortDefinition) throws BeansException {
		if (StringUtils.hasText(sortDefinition.getProperty())) {
			Collections.sort(source, new PropertyComparator<Object>(sortDefinition));
		}
	}

	public static void sort(Object[] source, SortDefinition sortDefinition) throws BeansException {
		if (StringUtils.hasText(sortDefinition.getProperty())) {
			Arrays.sort(source, new PropertyComparator<Object>(sortDefinition));
		}
	}

}

```

### PagedListHolder.java

```java
package org.springframework.beans.support;

import java.io.Serializable;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import org.springframework.util.Assert;

/**
 * 分页集合持有器
 */
@SuppressWarnings("serial")
public class PagedListHolder<E> implements Serializable {

	public static final int DEFAULT_PAGE_SIZE = 10;

	public static final int DEFAULT_MAX_LINKED_PAGES = 10;


	private List<E> source;

	private Date refreshDate;

	private SortDefinition sort;

	private SortDefinition sortUsed;

	private int pageSize = DEFAULT_PAGE_SIZE;

	private int page = 0;

	private boolean newPageSet;

	private int maxLinkedPages = DEFAULT_MAX_LINKED_PAGES;

	public PagedListHolder() {
		this(new ArrayList<E>(0));
	}

	public PagedListHolder(List<E> source) {
		this(source, new MutableSortDefinition(true));
	}

	public PagedListHolder(List<E> source, SortDefinition sort) {
		setSource(source);
		setSort(sort);
	}

	public void setSource(List<E> source) {
		Assert.notNull(source, "Source List must not be null");
		this.source = source;
		this.refreshDate = new Date();
		this.sortUsed = null;
	}

	public List<E> getSource() {
		return this.source;
	}

	public Date getRefreshDate() {
		return this.refreshDate;
	}

	public void setSort(SortDefinition sort) {
		this.sort = sort;
	}

	public SortDefinition getSort() {
		return this.sort;
	}

	public void setPageSize(int pageSize) {
		if (pageSize != this.pageSize) {
			this.pageSize = pageSize;
			if (!this.newPageSet) {
				this.page = 0;
			}
		}
	}

	public int getPageSize() {
		return this.pageSize;
	}

	public void setPage(int page) {
		this.page = page;
		this.newPageSet = true;
	}

	public int getPage() {
		this.newPageSet = false;
		if (this.page >= getPageCount()) {
			this.page = getPageCount() - 1;
		}
		return this.page;
	}

	public void setMaxLinkedPages(int maxLinkedPages) {
		this.maxLinkedPages = maxLinkedPages;
	}

	public int getMaxLinkedPages() {
		return this.maxLinkedPages;
	}

	public int getPageCount() {
		float nrOfPages = (float) getNrOfElements() / getPageSize();
		return (int) ((nrOfPages > (int) nrOfPages || nrOfPages == 0.0) ? nrOfPages + 1 : nrOfPages);
	}

	public boolean isFirstPage() {
		return getPage() == 0;
	}

	public boolean isLastPage() {
		return getPage() == getPageCount() -1;
	}

	public void previousPage() {
		if (!isFirstPage()) {
			this.page--;
		}
	}

	public void nextPage() {
		if (!isLastPage()) {
			this.page++;
		}
	}

	public int getNrOfElements() {
		return getSource().size();
	}

	public int getFirstElementOnPage() {
		return (getPageSize() * getPage());
	}

	public int getLastElementOnPage() {
		int endIndex = getPageSize() * (getPage() + 1);
		int size = getNrOfElements();
		return (endIndex > size ? size : endIndex) - 1;
	}

	public List<E> getPageList() {
		return getSource().subList(getFirstElementOnPage(), getLastElementOnPage() + 1);
	}

	public int getFirstLinkedPage() {
		return Math.max(0, getPage() - (getMaxLinkedPages() / 2));
	}

	public int getLastLinkedPage() {
		return Math.min(getFirstLinkedPage() + getMaxLinkedPages() - 1, getPageCount() - 1);
	}

	public void resort() {
		SortDefinition sort = getSort();
		if (sort != null && !sort.equals(this.sortUsed)) {
			this.sortUsed = copySortDefinition(sort);
			doSort(getSource(), sort);
			setPage(0);
		}
	}

	protected SortDefinition copySortDefinition(SortDefinition sort) {
		return new MutableSortDefinition(sort);
	}

	protected void doSort(List<E> source, SortDefinition sort) {
		PropertyComparator.sort(source, sort);
	}

}

```

### Demo

```java
package com.zby;

import java.util.Arrays;
import java.util.List;

import org.springframework.beans.support.MutableSortDefinition;
import org.springframework.beans.support.PagedListHolder;
import org.springframework.beans.support.SortDefinition;
import org.springframework.core.style.ToStringCreator;

public class RPagedListHolderMain {

	public static void main(String[] args) {
		List<User> list = Arrays.asList(new User(1L, "zby"), new User(2L, "aby"), new User(3L, "cby"), new User(4L, "pby"),
				new User(5L, "mby"), new User(6L, "iby"), new User(7L, "Qby"), new User(8L, "Cby"), new User(9L, "Pby"),
				new User(10L, "Mby"));
		System.out.println(list);
		System.out.println();
		SortDefinition sortDefinition = new MutableSortDefinition("name", true, true);
		PagedListHolder<User> pagedListHolder = new PagedListHolder<>(list, sortDefinition);
		pagedListHolder.resort();
		System.out.println(list);
		System.out.println();
		pagedListHolder.setPageSize(3);
		boolean over = false;
		while (!over) {
			if (pagedListHolder.isLastPage()) {
				over = true;
			}
			System.out.println(
					"第" + pagedListHolder.getPage() + "页,个数为" + pagedListHolder.getPageList().size() + ":" + pagedListHolder.getPageList());
			System.out.println();
			pagedListHolder.nextPage();
		}
	}

	static class User {
		private Long id;
		private String name;

		public User() {
			super();
		}

		public User(Long id, String name) {
			super();
			this.id = id;
			this.name = name;
		}

		public Long getId() {
			return id;
		}

		public void setId(Long id) {
			this.id = id;
		}

		public String getName() {
			return name;
		}

		public void setName(String name) {
			this.name = name;
		}

		@Override
		public String toString() {
			return new ToStringCreator(this).append("id", this.id).append("name", this.name).toString();
		}
	}
}

```

```console
[[RPagedListHolderMain.User@76ed5528 id = 1, name = 'zby'], [RPagedListHolderMain.User@2c7b84de id = 2, name = 'aby'], [RPagedListHolderMain.User@3fee733d id = 3, name = 'cby'], [RPagedListHolderMain.User@5acf9800 id = 4, name = 'pby'], [RPagedListHolderMain.User@4617c264 id = 5, name = 'mby'], [RPagedListHolderMain.User@36baf30c id = 6, name = 'iby'], [RPagedListHolderMain.User@7a81197d id = 7, name = 'Qby'], [RPagedListHolderMain.User@5ca881b5 id = 8, name = 'Cby'], [RPagedListHolderMain.User@24d46ca6 id = 9, name = 'Pby'], [RPagedListHolderMain.User@4517d9a3 id = 10, name = 'Mby']]

[[RPagedListHolderMain.User@2c7b84de id = 2, name = 'aby'], [RPagedListHolderMain.User@3fee733d id = 3, name = 'cby'], [RPagedListHolderMain.User@5ca881b5 id = 8, name = 'Cby'], [RPagedListHolderMain.User@36baf30c id = 6, name = 'iby'], [RPagedListHolderMain.User@4617c264 id = 5, name = 'mby'], [RPagedListHolderMain.User@4517d9a3 id = 10, name = 'Mby'], [RPagedListHolderMain.User@5acf9800 id = 4, name = 'pby'], [RPagedListHolderMain.User@24d46ca6 id = 9, name = 'Pby'], [RPagedListHolderMain.User@7a81197d id = 7, name = 'Qby'], [RPagedListHolderMain.User@76ed5528 id = 1, name = 'zby']]

第0页,个数为3:[[RPagedListHolderMain.User@2c7b84de id = 2, name = 'aby'], [RPagedListHolderMain.User@3fee733d id = 3, name = 'cby'], [RPagedListHolderMain.User@5ca881b5 id = 8, name = 'Cby']]

第1页,个数为3:[[RPagedListHolderMain.User@36baf30c id = 6, name = 'iby'], [RPagedListHolderMain.User@4617c264 id = 5, name = 'mby'], [RPagedListHolderMain.User@4517d9a3 id = 10, name = 'Mby']]

第2页,个数为3:[[RPagedListHolderMain.User@5acf9800 id = 4, name = 'pby'], [RPagedListHolderMain.User@24d46ca6 id = 9, name = 'Pby'], [RPagedListHolderMain.User@7a81197d id = 7, name = 'Qby']]

第3页,个数为1:[[RPagedListHolderMain.User@76ed5528 id = 1, name = 'zby']]


```

