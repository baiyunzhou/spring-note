# HttpMesssage

## HttpHeaders.java

```java
package org.springframework.http;
/**
 * HTTP请求或响应的头。headerName->valueList
 * http://www.w3.org/TR/cors/
 * http://tools.ietf.org/html/rfc7231#section-5.3.2
 */
public class HttpHeaders implements MultiValueMap<String, String>, Serializable {

	private static final long serialVersionUID = -8578554704772377436L;

	public static final String ACCEPT = "Accept";
	public static final String ACCEPT_CHARSET = "Accept-Charset";
	public static final String ACCEPT_ENCODING = "Accept-Encoding";
	public static final String ACCEPT_LANGUAGE = "Accept-Language";
	public static final String ACCEPT_RANGES = "Accept-Ranges";
	public static final String ACCESS_CONTROL_ALLOW_CREDENTIALS = "Access-Control-Allow-Credentials";
	public static final String ACCESS_CONTROL_ALLOW_HEADERS = "Access-Control-Allow-Headers";
	public static final String ACCESS_CONTROL_ALLOW_METHODS = "Access-Control-Allow-Methods";
	public static final String ACCESS_CONTROL_ALLOW_ORIGIN = "Access-Control-Allow-Origin";
	public static final String ACCESS_CONTROL_EXPOSE_HEADERS = "Access-Control-Expose-Headers";
	public static final String ACCESS_CONTROL_MAX_AGE = "Access-Control-Max-Age";
	public static final String ACCESS_CONTROL_REQUEST_HEADERS = "Access-Control-Request-Headers";
	public static final String ACCESS_CONTROL_REQUEST_METHOD = "Access-Control-Request-Method";
	public static final String AGE = "Age";
	public static final String ALLOW = "Allow";
	public static final String AUTHORIZATION = "Authorization";
	public static final String CACHE_CONTROL = "Cache-Control";
	public static final String CONNECTION = "Connection";
	public static final String CONTENT_ENCODING = "Content-Encoding";
	public static final String CONTENT_DISPOSITION = "Content-Disposition";
	public static final String CONTENT_LANGUAGE = "Content-Language";
	public static final String CONTENT_LENGTH = "Content-Length";
	public static final String CONTENT_LOCATION = "Content-Location";
	public static final String CONTENT_RANGE = "Content-Range";
	public static final String CONTENT_TYPE = "Content-Type";
	public static final String COOKIE = "Cookie";
	public static final String DATE = "Date";
	public static final String ETAG = "ETag";
	public static final String EXPECT = "Expect";
	public static final String EXPIRES = "Expires";
	public static final String FROM = "From";
	public static final String HOST = "Host";
	public static final String IF_MATCH = "If-Match";
	public static final String IF_MODIFIED_SINCE = "If-Modified-Since";
	public static final String IF_NONE_MATCH = "If-None-Match";
	public static final String IF_RANGE = "If-Range";
	public static final String IF_UNMODIFIED_SINCE = "If-Unmodified-Since";
	public static final String LAST_MODIFIED = "Last-Modified";
	public static final String LINK = "Link";
	public static final String LOCATION = "Location";
	public static final String MAX_FORWARDS = "Max-Forwards";
	public static final String ORIGIN = "Origin";
	public static final String PRAGMA = "Pragma";
	public static final String PROXY_AUTHENTICATE = "Proxy-Authenticate";
	public static final String PROXY_AUTHORIZATION = "Proxy-Authorization";
	public static final String RANGE = "Range";
	public static final String REFERER = "Referer";
	public static final String RETRY_AFTER = "Retry-After";
	public static final String SERVER = "Server";
	public static final String SET_COOKIE = "Set-Cookie";
	public static final String SET_COOKIE2 = "Set-Cookie2";
	public static final String TE = "TE";
	public static final String TRAILER = "Trailer";
	public static final String TRANSFER_ENCODING = "Transfer-Encoding";
	public static final String UPGRADE = "Upgrade";
	public static final String USER_AGENT = "User-Agent";
	public static final String VARY = "Vary";
	public static final String VIA = "Via";
	public static final String WARNING = "Warning";
	public static final String WWW_AUTHENTICATE = "WWW-Authenticate";

	private static final Pattern ETAG_HEADER_VALUE_PATTERN = Pattern.compile("\\*|\\s*((W\\/)?(\"[^\"]*\"))\\s*,?");

	private static final TimeZone GMT = TimeZone.getTimeZone("GMT");

	private static final String[] DATE_FORMATS = new String[] {
			"EEE, dd MMM yyyy HH:mm:ss zzz",
			"EEE, dd-MMM-yy HH:mm:ss zzz",
			"EEE MMM dd HH:mm:ss yyyy"
	};


	private final Map<String, List<String>> headers;

	public HttpHeaders() {
		this(new LinkedCaseInsensitiveMap<List<String>>(8, Locale.ENGLISH), false);
	}

	private HttpHeaders(Map<String, List<String>> headers, boolean readOnly) {
		if (readOnly) {
			Map<String, List<String>> map =
					new LinkedCaseInsensitiveMap<List<String>>(headers.size(), Locale.ENGLISH);
			for (Entry<String, List<String>> entry : headers.entrySet()) {
				List<String> values = Collections.unmodifiableList(entry.getValue());
				map.put(entry.getKey(), values);
			}
			this.headers = Collections.unmodifiableMap(map);
		}
		else {
			this.headers = headers;
		}
	}

	public void setAccept(List<MediaType> acceptableMediaTypes) {
		set(ACCEPT, MediaType.toString(acceptableMediaTypes));
	}

	public List<MediaType> getAccept() {
		return MediaType.parseMediaTypes(get(ACCEPT));
	}

	public void setAccessControlAllowCredentials(boolean allowCredentials) {
		set(ACCESS_CONTROL_ALLOW_CREDENTIALS, Boolean.toString(allowCredentials));
	}

	public boolean getAccessControlAllowCredentials() {
		return Boolean.parseBoolean(getFirst(ACCESS_CONTROL_ALLOW_CREDENTIALS));
	}

	public void setAccessControlAllowHeaders(List<String> allowedHeaders) {
		set(ACCESS_CONTROL_ALLOW_HEADERS, toCommaDelimitedString(allowedHeaders));
	}

	public List<String> getAccessControlAllowHeaders() {
		return getValuesAsList(ACCESS_CONTROL_ALLOW_HEADERS);
	}

	public void setAccessControlAllowMethods(List<HttpMethod> allowedMethods) {
		set(ACCESS_CONTROL_ALLOW_METHODS, StringUtils.collectionToCommaDelimitedString(allowedMethods));
	}

	public List<HttpMethod> getAccessControlAllowMethods() {
		List<HttpMethod> result = new ArrayList<HttpMethod>();
		String value = getFirst(ACCESS_CONTROL_ALLOW_METHODS);
		if (value != null) {
			String[] tokens = StringUtils.tokenizeToStringArray(value, ",");
			for (String token : tokens) {
				HttpMethod resolved = HttpMethod.resolve(token);
				if (resolved != null) {
					result.add(resolved);
				}
			}
		}
		return result;
	}

	public void setAccessControlAllowOrigin(String allowedOrigin) {
		set(ACCESS_CONTROL_ALLOW_ORIGIN, allowedOrigin);
	}

	public String getAccessControlAllowOrigin() {
		return getFieldValues(ACCESS_CONTROL_ALLOW_ORIGIN);
	}

	public void setAccessControlExposeHeaders(List<String> exposedHeaders) {
		set(ACCESS_CONTROL_EXPOSE_HEADERS, toCommaDelimitedString(exposedHeaders));
	}

	public List<String> getAccessControlExposeHeaders() {
		return getValuesAsList(ACCESS_CONTROL_EXPOSE_HEADERS);
	}

	public void setAccessControlMaxAge(long maxAge) {
		set(ACCESS_CONTROL_MAX_AGE, Long.toString(maxAge));
	}

	public long getAccessControlMaxAge() {
		String value = getFirst(ACCESS_CONTROL_MAX_AGE);
		return (value != null ? Long.parseLong(value) : -1);
	}

	public void setAccessControlRequestHeaders(List<String> requestHeaders) {
		set(ACCESS_CONTROL_REQUEST_HEADERS, toCommaDelimitedString(requestHeaders));
	}

	public List<String> getAccessControlRequestHeaders() {
		return getValuesAsList(ACCESS_CONTROL_REQUEST_HEADERS);
	}

	public void setAccessControlRequestMethod(HttpMethod requestMethod) {
		set(ACCESS_CONTROL_REQUEST_METHOD, requestMethod.name());
	}

	public HttpMethod getAccessControlRequestMethod() {
		return HttpMethod.resolve(getFirst(ACCESS_CONTROL_REQUEST_METHOD));
	}

	public void setAcceptCharset(List<Charset> acceptableCharsets) {
		StringBuilder builder = new StringBuilder();
		for (Iterator<Charset> iterator = acceptableCharsets.iterator(); iterator.hasNext();) {
			Charset charset = iterator.next();
			builder.append(charset.name().toLowerCase(Locale.ENGLISH));
			if (iterator.hasNext()) {
				builder.append(", ");
			}
		}
		set(ACCEPT_CHARSET, builder.toString());
	}

	public List<Charset> getAcceptCharset() {
		String value = getFirst(ACCEPT_CHARSET);
		if (value != null) {
			String[] tokens = StringUtils.tokenizeToStringArray(value, ",");
			List<Charset> result = new ArrayList<Charset>(tokens.length);
			for (String token : tokens) {
				int paramIdx = token.indexOf(';');
				String charsetName;
				if (paramIdx == -1) {
					charsetName = token;
				}
				else {
					charsetName = token.substring(0, paramIdx);
				}
				if (!charsetName.equals("*")) {
					result.add(Charset.forName(charsetName));
				}
			}
			return result;
		}
		else {
			return Collections.emptyList();
		}
	}

	public void setAllow(Set<HttpMethod> allowedMethods) {
		set(ALLOW, StringUtils.collectionToCommaDelimitedString(allowedMethods));
	}

	public Set<HttpMethod> getAllow() {
		String value = getFirst(ALLOW);
		if (!StringUtils.isEmpty(value)) {
			String[] tokens = StringUtils.tokenizeToStringArray(value, ",");
			List<HttpMethod> result = new ArrayList<HttpMethod>(tokens.length);
			for (String token : tokens) {
				HttpMethod resolved = HttpMethod.resolve(token);
				if (resolved != null) {
					result.add(resolved);
				}
			}
			return EnumSet.copyOf(result);
		}
		else {
			return EnumSet.noneOf(HttpMethod.class);
		}
	}

	public void setCacheControl(String cacheControl) {
		set(CACHE_CONTROL, cacheControl);
	}

	public String getCacheControl() {
		return getFieldValues(CACHE_CONTROL);
	}

	public void setConnection(String connection) {
		set(CONNECTION, connection);
	}

	public void setConnection(List<String> connection) {
		set(CONNECTION, toCommaDelimitedString(connection));
	}

	public List<String> getConnection() {
		return getValuesAsList(CONNECTION);
	}

	public void setContentDispositionFormData(String name, String filename) {
		Assert.notNull(name, "'name' must not be null");
		StringBuilder builder = new StringBuilder("form-data; name=\"");
		builder.append(name).append('\"');
		if (filename != null) {
			builder.append("; filename=\"");
			builder.append(filename).append('\"');
		}
		set(CONTENT_DISPOSITION, builder.toString());
	}

	@Deprecated
	public void setContentDispositionFormData(String name, String filename, Charset charset) {
		Assert.notNull(name, "'name' must not be null");
		StringBuilder builder = new StringBuilder("form-data; name=\"");
		builder.append(name).append('\"');
		if (filename != null) {
			if (charset == null || charset.name().equals("US-ASCII")) {
				builder.append("; filename=\"");
				builder.append(filename).append('\"');
			}
			else {
				builder.append("; filename*=");
				builder.append(encodeHeaderFieldParam(filename, charset));
			}
		}
		set(CONTENT_DISPOSITION, builder.toString());
	}

	public void setContentLength(long contentLength) {
		set(CONTENT_LENGTH, Long.toString(contentLength));
	}

	public long getContentLength() {
		String value = getFirst(CONTENT_LENGTH);
		return (value != null ? Long.parseLong(value) : -1);
	}

	public void setContentType(MediaType mediaType) {
		Assert.isTrue(!mediaType.isWildcardType(), "'Content-Type' cannot contain wildcard type '*'");
		Assert.isTrue(!mediaType.isWildcardSubtype(), "'Content-Type' cannot contain wildcard subtype '*'");
		set(CONTENT_TYPE, mediaType.toString());
	}

	public MediaType getContentType() {
		String value = getFirst(CONTENT_TYPE);
		return (StringUtils.hasLength(value) ? MediaType.parseMediaType(value) : null);
	}

	public void setDate(long date) {
		setDate(DATE, date);
	}

	public long getDate() {
		return getFirstDate(DATE);
	}

	public void setETag(String etag) {
		if (etag != null) {
			Assert.isTrue(etag.startsWith("\"") || etag.startsWith("W/"),
					"Invalid ETag: does not start with W/ or \"");
			Assert.isTrue(etag.endsWith("\""), "Invalid ETag: does not end with \"");
		}
		set(ETAG, etag);
	}

	public String getETag() {
		return getFirst(ETAG);
	}

	public void setExpires(long expires) {
		setDate(EXPIRES, expires);
	}

	public long getExpires() {
		return getFirstDate(EXPIRES, false);
	}

	public void setIfMatch(String ifMatch) {
		set(IF_MATCH, ifMatch);
	}

	public void setIfMatch(List<String> ifMatchList) {
		set(IF_MATCH, toCommaDelimitedString(ifMatchList));
	}

	public List<String> getIfMatch() {
		return getETagValuesAsList(IF_MATCH);
	}

	public void setIfModifiedSince(long ifModifiedSince) {
		setDate(IF_MODIFIED_SINCE, ifModifiedSince);
	}

	public long getIfModifiedSince() {
		return getFirstDate(IF_MODIFIED_SINCE, false);
	}

	public void setIfNoneMatch(String ifNoneMatch) {
		set(IF_NONE_MATCH, ifNoneMatch);
	}

	public void setIfNoneMatch(List<String> ifNoneMatchList) {
		set(IF_NONE_MATCH, toCommaDelimitedString(ifNoneMatchList));
	}

	public List<String> getIfNoneMatch() {
		return getETagValuesAsList(IF_NONE_MATCH);
	}

	public void setIfUnmodifiedSince(long ifUnmodifiedSince) {
		setDate(IF_UNMODIFIED_SINCE, ifUnmodifiedSince);
	}

	public long getIfUnmodifiedSince() {
		return getFirstDate(IF_UNMODIFIED_SINCE, false);
	}

	public void setLastModified(long lastModified) {
		setDate(LAST_MODIFIED, lastModified);
	}

	public long getLastModified() {
		return getFirstDate(LAST_MODIFIED, false);
	}

	public void setLocation(URI location) {
		set(LOCATION, location.toASCIIString());
	}

	public URI getLocation() {
		String value = getFirst(LOCATION);
		return (value != null ? URI.create(value) : null);
	}

	public void setOrigin(String origin) {
		set(ORIGIN, origin);
	}

	public String getOrigin() {
		return getFirst(ORIGIN);
	}

	public void setPragma(String pragma) {
		set(PRAGMA, pragma);
	}

	public String getPragma() {
		return getFirst(PRAGMA);
	}

	public void setRange(List<HttpRange> ranges) {
		String value = HttpRange.toString(ranges);
		set(RANGE, value);
	}

	public List<HttpRange> getRange() {
		String value = getFirst(RANGE);
		return HttpRange.parseRanges(value);
	}

	public void setUpgrade(String upgrade) {
		set(UPGRADE, upgrade);
	}

	public String getUpgrade() {
		return getFirst(UPGRADE);
	}

	public void setVary(List<String> requestHeaders) {
		set(VARY, toCommaDelimitedString(requestHeaders));
	}

	public List<String> getVary() {
		return getValuesAsList(VARY);
	}

	public void setDate(String headerName, long date) {
		SimpleDateFormat dateFormat = new SimpleDateFormat(DATE_FORMATS[0], Locale.US);
		dateFormat.setTimeZone(GMT);
		set(headerName, dateFormat.format(new Date(date)));
	}

	public long getFirstDate(String headerName) {
		return getFirstDate(headerName, true);
	}

	private long getFirstDate(String headerName, boolean rejectInvalid) {
		String headerValue = getFirst(headerName);
		if (headerValue == null) {
			// No header value sent at all
			return -1;
		}
		if (headerValue.length() >= 3) {
			// Short "0" or "-1" like values are never valid HTTP date headers...
			// Let's only bother with SimpleDateFormat parsing for long enough values.
			for (String dateFormat : DATE_FORMATS) {
				SimpleDateFormat simpleDateFormat = new SimpleDateFormat(dateFormat, Locale.US);
				simpleDateFormat.setTimeZone(GMT);
				try {
					return simpleDateFormat.parse(headerValue).getTime();
				}
				catch (ParseException ex) {
					// ignore
				}
			}
		}
		if (rejectInvalid) {
			throw new IllegalArgumentException("Cannot parse date value \"" + headerValue +
					"\" for \"" + headerName + "\" header");
		}
		return -1;
	}

	public List<String> getValuesAsList(String headerName) {
		List<String> values = get(headerName);
		if (values != null) {
			List<String> result = new ArrayList<String>();
			for (String value : values) {
				if (value != null) {
					String[] tokens = StringUtils.tokenizeToStringArray(value, ",");
					for (String token : tokens) {
						result.add(token);
					}
				}
			}
			return result;
		}
		return Collections.emptyList();
	}

	protected List<String> getETagValuesAsList(String headerName) {
		List<String> values = get(headerName);
		if (values != null) {
			List<String> result = new ArrayList<String>();
			for (String value : values) {
				if (value != null) {
					Matcher matcher = ETAG_HEADER_VALUE_PATTERN.matcher(value);
					while (matcher.find()) {
						if ("*".equals(matcher.group())) {
							result.add(matcher.group());
						}
						else {
							result.add(matcher.group(1));
						}
					}
					if (result.isEmpty()) {
						throw new IllegalArgumentException(
								"Could not parse header '" + headerName + "' with value '" + value + "'");
					}
				}
			}
			return result;
		}
		return Collections.emptyList();
	}

	protected String getFieldValues(String headerName) {
		List<String> headerValues = get(headerName);
		return (headerValues != null ? toCommaDelimitedString(headerValues) : null);
	}

	protected String toCommaDelimitedString(List<String> headerValues) {
		StringBuilder builder = new StringBuilder();
		for (Iterator<String> it = headerValues.iterator(); it.hasNext(); ) {
			String val = it.next();
			builder.append(val);
			if (it.hasNext()) {
				builder.append(", ");
			}
		}
		return builder.toString();
	}

	@Override
	public String getFirst(String headerName) {
		List<String> headerValues = this.headers.get(headerName);
		return (headerValues != null ? headerValues.get(0) : null);
	}

	@Override
	public void add(String headerName, String headerValue) {
		List<String> headerValues = this.headers.get(headerName);
		if (headerValues == null) {
			headerValues = new LinkedList<String>();
			this.headers.put(headerName, headerValues);
		}
		headerValues.add(headerValue);
	}

	@Override
	public void set(String headerName, String headerValue) {
		List<String> headerValues = new LinkedList<String>();
		headerValues.add(headerValue);
		this.headers.put(headerName, headerValues);
	}

	@Override
	public void setAll(Map<String, String> values) {
		for (Entry<String, String> entry : values.entrySet()) {
			set(entry.getKey(), entry.getValue());
		}
	}

	@Override
	public Map<String, String> toSingleValueMap() {
		LinkedHashMap<String, String> singleValueMap = new LinkedHashMap<String,String>(this.headers.size());
		for (Entry<String, List<String>> entry : this.headers.entrySet()) {
			singleValueMap.put(entry.getKey(), entry.getValue().get(0));
		}
		return singleValueMap;
	}


	// Map implementation

	@Override
	public int size() {
		return this.headers.size();
	}

	@Override
	public boolean isEmpty() {
		return this.headers.isEmpty();
	}

	@Override
	public boolean containsKey(Object key) {
		return this.headers.containsKey(key);
	}

	@Override
	public boolean containsValue(Object value) {
		return this.headers.containsValue(value);
	}

	@Override
	public List<String> get(Object key) {
		return this.headers.get(key);
	}

	@Override
	public List<String> put(String key, List<String> value) {
		return this.headers.put(key, value);
	}

	@Override
	public List<String> remove(Object key) {
		return this.headers.remove(key);
	}

	@Override
	public void putAll(Map<? extends String, ? extends List<String>> map) {
		this.headers.putAll(map);
	}

	@Override
	public void clear() {
		this.headers.clear();
	}

	@Override
	public Set<String> keySet() {
		return this.headers.keySet();
	}

	@Override
	public Collection<List<String>> values() {
		return this.headers.values();
	}

	@Override
	public Set<Entry<String, List<String>>> entrySet() {
		return this.headers.entrySet();
	}


	@Override
	public boolean equals(Object other) {
		if (this == other) {
			return true;
		}
		if (!(other instanceof HttpHeaders)) {
			return false;
		}
		HttpHeaders otherHeaders = (HttpHeaders) other;
		return this.headers.equals(otherHeaders.headers);
	}

	@Override
	public int hashCode() {
		return this.headers.hashCode();
	}

	@Override
	public String toString() {
		return this.headers.toString();
	}

	public static HttpHeaders readOnlyHttpHeaders(HttpHeaders headers) {
		Assert.notNull(headers, "HttpHeaders must not be null");
		return new HttpHeaders(headers, true);
	}

	static String encodeHeaderFieldParam(String input, Charset charset) {
		Assert.notNull(input, "Input String should not be null");
		Assert.notNull(charset, "Charset should not be null");
		if (charset.name().equals("US-ASCII")) {
			return input;
		}
		Assert.isTrue(charset.name().equals("UTF-8") || charset.name().equals("ISO-8859-1"),
				"Charset should be UTF-8 or ISO-8859-1");
		byte[] source = input.getBytes(charset);
		int len = source.length;
		StringBuilder sb = new StringBuilder(len << 1);
		sb.append(charset.name());
		sb.append("''");
		for (byte b : source) {
			if (isRFC5987AttrChar(b)) {
				sb.append((char) b);
			}
			else {
				sb.append('%');
				char hex1 = Character.toUpperCase(Character.forDigit((b >> 4) & 0xF, 16));
				char hex2 = Character.toUpperCase(Character.forDigit(b & 0xF, 16));
				sb.append(hex1);
				sb.append(hex2);
			}
		}
		return sb.toString();
	}

	private static boolean isRFC5987AttrChar(byte c) {
		return (c >= '0' && c <= '9') || (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z') ||
				c == '!' || c == '#' || c == '$' || c == '&' || c == '+' || c == '-' ||
				c == '.' || c == '^' || c == '_' || c == '`' || c == '|' || c == '~';
	}

}

```

## HttpMethod.java

```java
package org.springframework.http;

import java.util.HashMap;
import java.util.Map;

/**
 * HTTP方法枚举
 */
public enum HttpMethod {

	GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE;


	private static final Map<String, HttpMethod> mappings = new HashMap<String, HttpMethod>(16);

	static {
		for (HttpMethod httpMethod : values()) {
			mappings.put(httpMethod.name(), httpMethod);
		}
	}

	public static HttpMethod resolve(String method) {
		return (method != null ? mappings.get(method) : null);
	}

	public boolean matches(String method) {
		return (this == resolve(method));
	}

}

```

## HttpStatus.java

```java
package org.springframework.http;

public enum HttpStatus {
	
  	// 1xx Informational
	CONTINUE(100, "Continue"),
	SWITCHING_PROTOCOLS(101, "Switching Protocols"),
	PROCESSING(102, "Processing"),
	CHECKPOINT(103, "Checkpoint"),
    // 2xx Success
	OK(200, "OK"),
	CREATED(201, "Created"),
	ACCEPTED(202, "Accepted"),
	NON_AUTHORITATIVE_INFORMATION(203, "Non-Authoritative Information"),
	NO_CONTENT(204, "No Content"),
	RESET_CONTENT(205, "Reset Content"),
	PARTIAL_CONTENT(206, "Partial Content"),
	MULTI_STATUS(207, "Multi-Status"),
	ALREADY_REPORTED(208, "Already Reported"),
	IM_USED(226, "IM Used"),
	// 3xx Redirection
	MULTIPLE_CHOICES(300, "Multiple Choices"),
	MOVED_PERMANENTLY(301, "Moved Permanently"),
	FOUND(302, "Found"),
	@Deprecated
	MOVED_TEMPORARILY(302, "Moved Temporarily"),
	SEE_OTHER(303, "See Other"),
	NOT_MODIFIED(304, "Not Modified"),
	@Deprecated
	USE_PROXY(305, "Use Proxy"),
	TEMPORARY_REDIRECT(307, "Temporary Redirect"),
	PERMANENT_REDIRECT(308, "Permanent Redirect"),
	// --- 4xx Client Error ---
	BAD_REQUEST(400, "Bad Request"),
	UNAUTHORIZED(401, "Unauthorized"),
	PAYMENT_REQUIRED(402, "Payment Required"),
	FORBIDDEN(403, "Forbidden"),
	NOT_FOUND(404, "Not Found"),
	METHOD_NOT_ALLOWED(405, "Method Not Allowed"),
	NOT_ACCEPTABLE(406, "Not Acceptable"),
	PROXY_AUTHENTICATION_REQUIRED(407, "Proxy Authentication Required"),
	REQUEST_TIMEOUT(408, "Request Timeout"),
	CONFLICT(409, "Conflict"),
	GONE(410, "Gone"),
	LENGTH_REQUIRED(411, "Length Required"),
	PRECONDITION_FAILED(412, "Precondition Failed"),
	PAYLOAD_TOO_LARGE(413, "Payload Too Large"),
	@Deprecated
	REQUEST_ENTITY_TOO_LARGE(413, "Request Entity Too Large"),
	URI_TOO_LONG(414, "URI Too Long"),
	@Deprecated
	REQUEST_URI_TOO_LONG(414, "Request-URI Too Long"),
	UNSUPPORTED_MEDIA_TYPE(415, "Unsupported Media Type"),
	REQUESTED_RANGE_NOT_SATISFIABLE(416, "Requested range not satisfiable"),
	EXPECTATION_FAILED(417, "Expectation Failed"),
	I_AM_A_TEAPOT(418, "I'm a teapot"),
	@Deprecated
	INSUFFICIENT_SPACE_ON_RESOURCE(419, "Insufficient Space On Resource"),
	@Deprecated
	METHOD_FAILURE(420, "Method Failure"),
	@Deprecated
	DESTINATION_LOCKED(421, "Destination Locked"),
	UNPROCESSABLE_ENTITY(422, "Unprocessable Entity"),
	LOCKED(423, "Locked"),
	FAILED_DEPENDENCY(424, "Failed Dependency"),
	UPGRADE_REQUIRED(426, "Upgrade Required"),
	PRECONDITION_REQUIRED(428, "Precondition Required"),
	TOO_MANY_REQUESTS(429, "Too Many Requests"),
	REQUEST_HEADER_FIELDS_TOO_LARGE(431, "Request Header Fields Too Large"),
	UNAVAILABLE_FOR_LEGAL_REASONS(451, "Unavailable For Legal Reasons"),

	// --- 5xx Server Error ---
	INTERNAL_SERVER_ERROR(500, "Internal Server Error"),
	NOT_IMPLEMENTED(501, "Not Implemented"),
	BAD_GATEWAY(502, "Bad Gateway"),
	SERVICE_UNAVAILABLE(503, "Service Unavailable"),
	GATEWAY_TIMEOUT(504, "Gateway Timeout"),
	HTTP_VERSION_NOT_SUPPORTED(505, "HTTP Version not supported"),
	VARIANT_ALSO_NEGOTIATES(506, "Variant Also Negotiates"),
	INSUFFICIENT_STORAGE(507, "Insufficient Storage"),
	LOOP_DETECTED(508, "Loop Detected"),
	BANDWIDTH_LIMIT_EXCEEDED(509, "Bandwidth Limit Exceeded"),
	NOT_EXTENDED(510, "Not Extended"),
	NETWORK_AUTHENTICATION_REQUIRED(511, "Network Authentication Required");


	private final int value;

	private final String reasonPhrase;


	HttpStatus(int value, String reasonPhrase) {
		this.value = value;
		this.reasonPhrase = reasonPhrase;
	}
	public int value() {
		return this.value;
	}
	public String getReasonPhrase() {
		return this.reasonPhrase;
	}

	public boolean is1xxInformational() {
		return Series.INFORMATIONAL.equals(series());
	}

	public boolean is2xxSuccessful() {
		return Series.SUCCESSFUL.equals(series());
	}

	public boolean is3xxRedirection() {
		return Series.REDIRECTION.equals(series());
	}

	public boolean is4xxClientError() {
		return Series.CLIENT_ERROR.equals(series());
	}

	public boolean is5xxServerError() {
		return Series.SERVER_ERROR.equals(series());
	}

	public Series series() {
		return Series.valueOf(this);
	}

	@Override
	public String toString() {
		return Integer.toString(this.value);
	}

	public static HttpStatus valueOf(int statusCode) {
		for (HttpStatus status : values()) {
			if (status.value == statusCode) {
				return status;
			}
		}
		throw new IllegalArgumentException("No matching constant for [" + statusCode + "]");
	}

	public enum Series {

		INFORMATIONAL(1),
		SUCCESSFUL(2),
		REDIRECTION(3),
		CLIENT_ERROR(4),
		SERVER_ERROR(5);

		private final int value;

		Series(int value) {
			this.value = value;
		}

		public int value() {
			return this.value;
		}

		public static Series valueOf(int status) {
			int seriesCode = status / 100;
			for (Series series : values()) {
				if (series.value == seriesCode) {
					return series;
				}
			}
			throw new IllegalArgumentException("No matching constant for [" + status + "]");
		}

		public static Series valueOf(HttpStatus status) {
			return valueOf(status.value);
		}
	}

}
```
## HttpMessage.java

```java
package org.springframework.http;

/**
 * HTTP请求或响应的基本接口
 */
public interface HttpMessage {

	HttpHeaders getHeaders();

}

```

### HttpInputMessage.java

```java
package org.springframework.http;

/**
 * HTTP请求
 */
public interface HttpInputMessage extends HttpMessage {

	InputStream getBody() throws IOException;

}

```

#### MappingJacksonInputMessage.java

```java
package org.springframework.http.converter.json;
/**
 * 能存储Jackson视图
 */
public class MappingJacksonInputMessage implements HttpInputMessage {

	private final InputStream body;

	private final HttpHeaders headers;

	private Class<?> deserializationView;


	public MappingJacksonInputMessage(InputStream body, HttpHeaders headers) {
		this.body = body;
		this.headers = headers;
	}

	public MappingJacksonInputMessage(InputStream body, HttpHeaders headers, Class<?> deserializationView) {
		this(body, headers);
		this.deserializationView = deserializationView;
	}


	@Override
	public InputStream getBody() throws IOException {
		return this.body;
	}

	@Override
	public HttpHeaders getHeaders() {
		return this.headers;
	}

	public void setDeserializationView(Class<?> deserializationView) {
		this.deserializationView = deserializationView;
	}

	public Class<?> getDeserializationView() {
		return this.deserializationView;
	}

}

```



### HttpOutputMessage.java

```java
package org.springframework.http;
/**
 * HTTP响应
 */
public interface HttpOutputMessage extends HttpMessage {

	OutputStream getBody() throws IOException;

}

```

#### ServerHttpResponse.java

```java
package org.springframework.http.server;
/**
 * 代表服务端HTTP响应
 */
public interface ServerHttpResponse extends HttpOutputMessage, Flushable, Closeable {

	void setStatusCode(HttpStatus status);

	@Override
	void flush() throws IOException;

	@Override
	void close();

}

```

##### ServletServerHttpResponse.java

```java
package org.springframework.http.server;
/**
 * Servlet实现
 */
public class ServletServerHttpResponse implements ServerHttpResponse {

	private static final boolean servlet3Present =
			ClassUtils.hasMethod(HttpServletResponse.class, "getHeader", String.class);


	private final HttpServletResponse servletResponse;

	private final HttpHeaders headers;

	private boolean headersWritten = false;

	private boolean bodyUsed = false;


	public ServletServerHttpResponse(HttpServletResponse servletResponse) {
		Assert.notNull(servletResponse, "HttpServletResponse must not be null");
		this.servletResponse = servletResponse;
		this.headers = (servlet3Present ? new ServletResponseHttpHeaders() : new HttpHeaders());
	}

	public HttpServletResponse getServletResponse() {
		return this.servletResponse;
	}

	@Override
	public void setStatusCode(HttpStatus status) {
		Assert.notNull(status, "HttpStatus must not be null");
		this.servletResponse.setStatus(status.value());
	}

	@Override
	public HttpHeaders getHeaders() {
		return (this.headersWritten ? HttpHeaders.readOnlyHttpHeaders(this.headers) : this.headers);
	}

	@Override
	public OutputStream getBody() throws IOException {
		this.bodyUsed = true;
		writeHeaders();
		return this.servletResponse.getOutputStream();
	}

	@Override
	public void flush() throws IOException {
		writeHeaders();
		if (this.bodyUsed) {
			this.servletResponse.flushBuffer();
		}
	}

	@Override
	public void close() {
		writeHeaders();
	}

	private void writeHeaders() {
		if (!this.headersWritten) {
			for (Map.Entry<String, List<String>> entry : this.headers.entrySet()) {
				String headerName = entry.getKey();
				for (String headerValue : entry.getValue()) {
					this.servletResponse.addHeader(headerName, headerValue);
				}
			}
			// HttpServletResponse exposes some headers as properties: we should include those if not already present
			if (this.servletResponse.getContentType() == null && this.headers.getContentType() != null) {
				this.servletResponse.setContentType(this.headers.getContentType().toString());
			}
			if (this.servletResponse.getCharacterEncoding() == null && this.headers.getContentType() != null &&
					this.headers.getContentType().getCharset() != null) {
				this.servletResponse.setCharacterEncoding(this.headers.getContentType().getCharset().name());
			}
			this.headersWritten = true;
		}
	}

	private class ServletResponseHttpHeaders extends HttpHeaders {

		private static final long serialVersionUID = 3410708522401046302L;

		@Override
		public boolean containsKey(Object key) {
			return (super.containsKey(key) || (get(key) != null));
		}

		@Override
		public String getFirst(String headerName) {
			String value = servletResponse.getHeader(headerName);
			if (value != null) {
				return value;
			}
			else {
				return super.getFirst(headerName);
			}
		}

		@Override
		public List<String> get(Object key) {
			Assert.isInstanceOf(String.class, key, "Key must be a String-based header name");

			Collection<String> values1 = servletResponse.getHeaders((String) key);
			boolean isEmpty1 = CollectionUtils.isEmpty(values1);

			List<String> values2 = super.get(key);
			boolean isEmpty2 = CollectionUtils.isEmpty(values2);

			if (isEmpty1 && isEmpty2) {
				return null;
			}

			List<String> values = new ArrayList<String>();
			if (!isEmpty1) {
				values.addAll(values1);
			}
			if (!isEmpty2) {
				values.addAll(values2);
			}
			return values;
		}
	}

}

```



### HttpRequest.java

```java
package org.springframework.http;

/**
 * 代表HTTP请求
 */
public interface HttpRequest extends HttpMessage {

	HttpMethod getMethod();

	URI getURI();

}

```

#### ServerHttpRequest.java

```java
package org.springframework.http.server;

/**
 * 代表服务端的HTP请求
 */
public interface ServerHttpRequest extends HttpRequest, HttpInputMessage {

	Principal getPrincipal();

	InetSocketAddress getLocalAddress();

	InetSocketAddress getRemoteAddress();

	ServerHttpAsyncRequestControl getAsyncRequestControl(ServerHttpResponse response);

}

```

##### ServletServerHttpRequest.java

```java
package org.springframework.http.server;
/**
 * Servlet实现
 */
public class ServletServerHttpRequest implements ServerHttpRequest {

	protected static final String FORM_CONTENT_TYPE = "application/x-www-form-urlencoded";

	protected static final String FORM_CHARSET = "UTF-8";


	private final HttpServletRequest servletRequest;

	private URI uri;

	private HttpHeaders headers;

	private ServerHttpAsyncRequestControl asyncRequestControl;

	public ServletServerHttpRequest(HttpServletRequest servletRequest) {
		Assert.notNull(servletRequest, "HttpServletRequest must not be null");
		this.servletRequest = servletRequest;
	}

	public HttpServletRequest getServletRequest() {
		return this.servletRequest;
	}

	@Override
	public HttpMethod getMethod() {
		return HttpMethod.resolve(this.servletRequest.getMethod());
	}

	@Override
	public URI getURI() {
		if (this.uri == null) {
			String urlString = null;
			boolean hasQuery = false;
			try {
				StringBuffer url = this.servletRequest.getRequestURL();
				String query = this.servletRequest.getQueryString();
				hasQuery = StringUtils.hasText(query);
				if (hasQuery) {
					url.append('?').append(query);
				}
				urlString = url.toString();
				this.uri = new URI(urlString);
			}
			catch (URISyntaxException ex) {
				if (!hasQuery) {
					throw new IllegalStateException(
							"Could not resolve HttpServletRequest as URI: " + urlString, ex);
				}
				// Maybe a malformed query string... try plain request URL
				try {
					urlString = this.servletRequest.getRequestURL().toString();
					this.uri = new URI(urlString);
				}
				catch (URISyntaxException ex2) {
					throw new IllegalStateException(
							"Could not resolve HttpServletRequest as URI: " + urlString, ex2);
				}
			}
		}
		return this.uri;
	}

	@Override
	public HttpHeaders getHeaders() {
		if (this.headers == null) {
			this.headers = new HttpHeaders();

			for (Enumeration<?> names = this.servletRequest.getHeaderNames(); names.hasMoreElements();) {
				String headerName = (String) names.nextElement();
				for (Enumeration<?> headerValues = this.servletRequest.getHeaders(headerName);
						headerValues.hasMoreElements();) {
					String headerValue = (String) headerValues.nextElement();
					this.headers.add(headerName, headerValue);
				}
			}
			try {
				MediaType contentType = this.headers.getContentType();
				if (contentType == null) {
					String requestContentType = this.servletRequest.getContentType();
					if (StringUtils.hasLength(requestContentType)) {
						contentType = MediaType.parseMediaType(requestContentType);
						this.headers.setContentType(contentType);
					}
				}
				if (contentType != null && contentType.getCharset() == null) {
					String requestEncoding = this.servletRequest.getCharacterEncoding();
					if (StringUtils.hasLength(requestEncoding)) {
						Charset charSet = Charset.forName(requestEncoding);
						Map<String, String> params = new LinkedCaseInsensitiveMap<String>();
						params.putAll(contentType.getParameters());
						params.put("charset", charSet.toString());
						MediaType mediaType = new MediaType(contentType.getType(), contentType.getSubtype(), params);
						this.headers.setContentType(mediaType);
					}
				}
			}
			catch (InvalidMediaTypeException ex) {
				// Ignore: simply not exposing an invalid content type in HttpHeaders...
			}

			if (this.headers.getContentLength() < 0) {
				int requestContentLength = this.servletRequest.getContentLength();
				if (requestContentLength != -1) {
					this.headers.setContentLength(requestContentLength);
				}
			}
		}

		return this.headers;
	}

	@Override
	public Principal getPrincipal() {
		return this.servletRequest.getUserPrincipal();
	}

	@Override
	public InetSocketAddress getLocalAddress() {
		return new InetSocketAddress(this.servletRequest.getLocalName(), this.servletRequest.getLocalPort());
	}

	@Override
	public InetSocketAddress getRemoteAddress() {
		return new InetSocketAddress(this.servletRequest.getRemoteHost(), this.servletRequest.getRemotePort());
	}

	@Override
	public InputStream getBody() throws IOException {
		if (isFormPost(this.servletRequest)) {
			return getBodyFromServletRequestParameters(this.servletRequest);
		}
		else {
			return this.servletRequest.getInputStream();
		}
	}

	@Override
	public ServerHttpAsyncRequestControl getAsyncRequestControl(ServerHttpResponse response) {
		if (this.asyncRequestControl == null) {
			if (!ServletServerHttpResponse.class.isInstance(response)) {
				throw new IllegalArgumentException(
						"Response must be a ServletServerHttpResponse: " + response.getClass());
			}
			ServletServerHttpResponse servletServerResponse = (ServletServerHttpResponse) response;
			this.asyncRequestControl = new ServletServerHttpAsyncRequestControl(this, servletServerResponse);
		}
		return this.asyncRequestControl;
	}


	private static boolean isFormPost(HttpServletRequest request) {
		String contentType = request.getContentType();
		return (contentType != null && contentType.contains(FORM_CONTENT_TYPE) &&
				HttpMethod.POST.matches(request.getMethod()));
	}

	private static InputStream getBodyFromServletRequestParameters(HttpServletRequest request) throws IOException {
		ByteArrayOutputStream bos = new ByteArrayOutputStream(1024);
		Writer writer = new OutputStreamWriter(bos, FORM_CHARSET);

		Map<String, String[]> form = request.getParameterMap();
		for (Iterator<String> nameIterator = form.keySet().iterator(); nameIterator.hasNext();) {
			String name = nameIterator.next();
			List<String> values = Arrays.asList(form.get(name));
			for (Iterator<String> valueIterator = values.iterator(); valueIterator.hasNext();) {
				String value = valueIterator.next();
				writer.write(URLEncoder.encode(name, FORM_CHARSET));
				if (value != null) {
					writer.write('=');
					writer.write(URLEncoder.encode(value, FORM_CHARSET));
					if (valueIterator.hasNext()) {
						writer.write('&');
					}
				}
			}
			if (nameIterator.hasNext()) {
				writer.append('&');
			}
		}
		writer.flush();

		return new ByteArrayInputStream(bos.toByteArray());
	}

}

```



# HttpMessageConverter

## HttpMessageConverter.java

```java
package org.springframework.http.converter;
/**
 * HTTP请求响应转换器
 */
public interface HttpMessageConverter<T> {

	boolean canRead(Class<?> clazz, MediaType mediaType);

	boolean canWrite(Class<?> clazz, MediaType mediaType);

	List<MediaType> getSupportedMediaTypes();

	T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;

	void write(T t, MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

}

```

### GenericHttpMessageConverter.java

```java
package org.springframework.http.converter;

import java.io.IOException;
import java.lang.reflect.Type;

import org.springframework.http.HttpInputMessage;
import org.springframework.http.HttpOutputMessage;
import org.springframework.http.MediaType;

/**
 * 
 */
public interface GenericHttpMessageConverter<T> extends HttpMessageConverter<T> {

	boolean canRead(Type type, Class<?> contextClass, MediaType mediaType);

	T read(Type type, Class<?> contextClass, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;

	boolean canWrite(Type type, Class<?> clazz, MediaType mediaType);

	void write(T t, Type type, MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

}

```

#### FastJsonHttpMessageConverter.java

```java
package com.alibaba.fastjson.support.spring;
/**
 * Fastjson转化Json
 */

public class FastJsonHttpMessageConverter extends AbstractHttpMessageConverter<Object>//
        implements GenericHttpMessageConverter<Object> {

    public static final MediaType APPLICATION_JAVASCRIPT = new MediaType("application", "javascript");

    private Charset charset = Charset.forName("UTF-8");

    @Deprecated
    protected SerializerFeature[] features = new SerializerFeature[0];

    @Deprecated
    protected SerializeFilter[] filters = new SerializeFilter[0];

    @Deprecated
    protected String dateFormat;

    /**
     * with fastJson config
     */
    private FastJsonConfig fastJsonConfig = new FastJsonConfig();

    /**
     * @return the fastJsonConfig.
     * @since 1.2.11
     */
    public FastJsonConfig getFastJsonConfig() {
        return fastJsonConfig;
    }

    /**
     * @param fastJsonConfig the fastJsonConfig to set.
     * @since 1.2.11
     */
    public void setFastJsonConfig(FastJsonConfig fastJsonConfig) {
        this.fastJsonConfig = fastJsonConfig;
    }

    /**
     * Can serialize/deserialize all types.
     */
    public FastJsonHttpMessageConverter() {

        super(MediaType.ALL);
    }

    @Deprecated
    public Charset getCharset() {
        return this.fastJsonConfig.getCharset();
    }

    @Deprecated
    public void setCharset(Charset charset) {
        this.fastJsonConfig.setCharset(charset);
    }

    @Deprecated
    public String getDateFormat() {
        return this.fastJsonConfig.getDateFormat();
    }

    @Deprecated
    public void setDateFormat(String dateFormat) {
        this.fastJsonConfig.setDateFormat(dateFormat);
    }

    @Deprecated
    public SerializerFeature[] getFeatures() {
        return this.fastJsonConfig.getSerializerFeatures();
    }

    @Deprecated
    public void setFeatures(SerializerFeature... features) {
        this.fastJsonConfig.setSerializerFeatures(features);
    }

    @Deprecated
    public SerializeFilter[] getFilters() {
        return this.fastJsonConfig.getSerializeFilters();
    }

    @Deprecated
    public void setFilters(SerializeFilter... filters) {
        this.fastJsonConfig.setSerializeFilters(filters);
    }

    @Deprecated
    public void addSerializeFilter(SerializeFilter filter) {
        if (filter == null) {
            return;
        }

        int length = this.fastJsonConfig.getSerializeFilters().length;
        SerializeFilter[] filters = new SerializeFilter[length + 1];
        System.arraycopy(this.fastJsonConfig.getSerializeFilters(), 0, filters, 0, length);
        filters[filters.length - 1] = filter;
        this.fastJsonConfig.setSerializeFilters(filters);
    }

    @Override
    protected boolean supports(Class<?> clazz) {
        return true;
    }


    public boolean canRead(Type type, Class<?> contextClass, MediaType mediaType) {
        return super.canRead(contextClass, mediaType);
    }


    public boolean canWrite(Type type, Class<?> clazz, MediaType mediaType) {
        return super.canWrite(clazz, mediaType);
    }

    /*
     * @see org.springframework.http.converter.GenericHttpMessageConverter#read(java.lang.reflect.Type, java.lang.Class, org.springframework.http.HttpInputMessage)
     */
    public Object read(Type type, //
                       Class<?> contextClass, //
                       HttpInputMessage inputMessage //
    ) throws IOException, HttpMessageNotReadableException {
        return readType(getType(type, contextClass), inputMessage);
    }

    /*
     * @see org.springframework.http.converter.GenericHttpMessageConverter.write
     */
    public void write(Object o, Type type, MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
        super.write(o, contentType, outputMessage);// support StreamingHttpOutputMessage in spring4.0+
        //writeInternal(o, outputMessage);
    }


    /*
     * @see org.springframework.http.converter.AbstractHttpMessageConverter#readInternal(java.lang.Class, org.springframework.http.HttpInputMessage)
     */
    @Override
    protected Object readInternal(Class<? extends Object> clazz, //
                                  HttpInputMessage inputMessage //
    ) throws IOException, HttpMessageNotReadableException {
        return readType(getType(clazz, null), inputMessage);
    }

    private Object readType(Type type, HttpInputMessage inputMessage) throws IOException {

        try {
            InputStream in = inputMessage.getBody();
            return JSON.parseObject(in, fastJsonConfig.getCharset(), type, fastJsonConfig.getFeatures());
        } catch (JSONException ex) {
            throw new HttpMessageNotReadableException("JSON parse error: " + ex.getMessage(), ex);
        } catch (IOException ex) {
            throw new HttpMessageNotReadableException("I/O error while reading input message", ex);
        }
    }

    @Override
    protected void writeInternal(Object object, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {

        ByteArrayOutputStream outnew = new ByteArrayOutputStream();
        try {
            HttpHeaders headers = outputMessage.getHeaders();

            //获取全局配置的filter
            SerializeFilter[] globalFilters = fastJsonConfig.getSerializeFilters();
            List<SerializeFilter> allFilters = new ArrayList<SerializeFilter>(Arrays.asList(globalFilters));

            boolean isJsonp = false;

            //不知道为什么会有这行代码， 但是为了保持和原来的行为一致，还是保留下来
            Object value = strangeCodeForJackson(object);

            if (value instanceof FastJsonContainer) {
                FastJsonContainer fastJsonContainer = (FastJsonContainer) value;
                PropertyPreFilters filters = fastJsonContainer.getFilters();
                allFilters.addAll(filters.getFilters());
                value = fastJsonContainer.getValue();
            }

            //revise 2017-10-23 ,
            // 保持原有的MappingFastJsonValue对象的contentType不做修改 保持旧版兼容。
            // 但是新的JSONPObject将返回标准的contentType：application/javascript ，不对是否有function进行判断
            if (value instanceof MappingFastJsonValue) {
                if(!StringUtils.isEmpty(((MappingFastJsonValue) value).getJsonpFunction())){
                    isJsonp = true;
                }
            } else if (value instanceof JSONPObject) {
                isJsonp = true;
            }


            int len = JSON.writeJSONString(outnew, //
                    fastJsonConfig.getCharset(), //
                    value, //
                    fastJsonConfig.getSerializeConfig(), //
                    //fastJsonConfig.getSerializeFilters(), //
                    allFilters.toArray(new SerializeFilter[allFilters.size()]),
                    fastJsonConfig.getDateFormat(), //
                    JSON.DEFAULT_GENERATE_FEATURE, //
                    fastJsonConfig.getSerializerFeatures());

            if (isJsonp) {
                headers.setContentType(APPLICATION_JAVASCRIPT);
            }

            if (fastJsonConfig.isWriteContentLength()) {
                headers.setContentLength(len);
            }

            outnew.writeTo(outputMessage.getBody());

        } catch (JSONException ex) {
            throw new HttpMessageNotWritableException("Could not write JSON: " + ex.getMessage(), ex);
        } finally {
            outnew.close();
        }
    }

    private Object strangeCodeForJackson(Object obj) {
        if (obj != null) {
            String className = obj.getClass().getName();
            if ("com.fasterxml.jackson.databind.node.ObjectNode".equals(className)) {
                return obj.toString();
            }
        }
        return obj;
    }

    protected Type getType(Type type, Class<?> contextClass) {
        if (Spring4TypeResolvableHelper.isSupport()) {
            return Spring4TypeResolvableHelper.getType(type, contextClass);
        }

        return type;
    }


    private static class Spring4TypeResolvableHelper {
        private static boolean hasClazzResolvableType;

        static {
            try {
                Class.forName("org.springframework.core.ResolvableType");
                hasClazzResolvableType = true;
            } catch (ClassNotFoundException e) {
                hasClazzResolvableType = false;
            }
        }

        private static boolean isSupport() {
            return hasClazzResolvableType;
        }


        private static Type getType(Type type, Class<?> contextClass) {
            if (contextClass != null) {
                ResolvableType resolvedType = ResolvableType.forType(type);
                if (type instanceof TypeVariable) {
                    ResolvableType resolvedTypeVariable = resolveVariable((TypeVariable) type, ResolvableType.forClass(contextClass));
                    if (resolvedTypeVariable != ResolvableType.NONE) {
                        return resolvedTypeVariable.resolve();
                    }
                } else if (type instanceof ParameterizedType && resolvedType.hasUnresolvableGenerics()) {
                    ParameterizedType parameterizedType = (ParameterizedType) type;
                    Class<?>[] generics = new Class[parameterizedType.getActualTypeArguments().length];
                    Type[] typeArguments = parameterizedType.getActualTypeArguments();

                    for (int i = 0; i < typeArguments.length; ++i) {
                        Type typeArgument = typeArguments[i];
                        if (typeArgument instanceof TypeVariable) {
                            ResolvableType resolvedTypeArgument = resolveVariable((TypeVariable) typeArgument, ResolvableType.forClass(contextClass));
                            if (resolvedTypeArgument != ResolvableType.NONE) {
                                generics[i] = resolvedTypeArgument.resolve();
                            } else {
                                generics[i] = ResolvableType.forType(typeArgument).resolve();
                            }
                        } else {
                            generics[i] = ResolvableType.forType(typeArgument).resolve();
                        }
                    }

                    return ResolvableType.forClassWithGenerics(resolvedType.getRawClass(), generics).getType();
                }
            }

            return type;
        }

        private static ResolvableType resolveVariable(TypeVariable<?> typeVariable, ResolvableType contextType) {
            ResolvableType resolvedType;
            if (contextType.hasGenerics()) {
                resolvedType = ResolvableType.forType(typeVariable, contextType);
                if (resolvedType.resolve() != null) {
                    return resolvedType;
                }
            }

            ResolvableType superType = contextType.getSuperType();
            if (superType != ResolvableType.NONE) {
                resolvedType = resolveVariable(typeVariable, superType);
                if (resolvedType.resolve() != null) {
                    return resolvedType;
                }
            }
            for (ResolvableType ifc : contextType.getInterfaces()) {
                resolvedType = resolveVariable(typeVariable, ifc);
                if (resolvedType.resolve() != null) {
                    return resolvedType;
                }
            }
            return ResolvableType.NONE;
        }
    }


}

```



### AbstractHttpMessageConverter.java

```java
package org.springframework.http.converter;
/**
 * 抽象实现
 */
public abstract class AbstractHttpMessageConverter<T> implements HttpMessageConverter<T> {

	protected final Log logger = LogFactory.getLog(getClass());

	private List<MediaType> supportedMediaTypes = Collections.emptyList();

	private Charset defaultCharset;

	protected AbstractHttpMessageConverter() {
	}

	protected AbstractHttpMessageConverter(MediaType supportedMediaType) {
		setSupportedMediaTypes(Collections.singletonList(supportedMediaType));
	}

	protected AbstractHttpMessageConverter(MediaType... supportedMediaTypes) {
		setSupportedMediaTypes(Arrays.asList(supportedMediaTypes));
	}

	protected AbstractHttpMessageConverter(Charset defaultCharset, MediaType... supportedMediaTypes) {
		this.defaultCharset = defaultCharset;
		setSupportedMediaTypes(Arrays.asList(supportedMediaTypes));
	}

	public void setSupportedMediaTypes(List<MediaType> supportedMediaTypes) {
		Assert.notEmpty(supportedMediaTypes, "MediaType List must not be empty");
		this.supportedMediaTypes = new ArrayList<MediaType>(supportedMediaTypes);
	}

	@Override
	public List<MediaType> getSupportedMediaTypes() {
		return Collections.unmodifiableList(this.supportedMediaTypes);
	}

	public void setDefaultCharset(Charset defaultCharset) {
		this.defaultCharset = defaultCharset;
	}

	public Charset getDefaultCharset() {
		return this.defaultCharset;
	}

	@Override
	public boolean canRead(Class<?> clazz, MediaType mediaType) {
		return supports(clazz) && canRead(mediaType);
	}

	protected boolean canRead(MediaType mediaType) {
		if (mediaType == null) {
			return true;
		}
		for (MediaType supportedMediaType : getSupportedMediaTypes()) {
			if (supportedMediaType.includes(mediaType)) {
				return true;
			}
		}
		return false;
	}

	@Override
	public boolean canWrite(Class<?> clazz, MediaType mediaType) {
		return supports(clazz) && canWrite(mediaType);
	}

	protected boolean canWrite(MediaType mediaType) {
		if (mediaType == null || MediaType.ALL.equals(mediaType)) {
			return true;
		}
		for (MediaType supportedMediaType : getSupportedMediaTypes()) {
			if (supportedMediaType.isCompatibleWith(mediaType)) {
				return true;
			}
		}
		return false;
	}

	@Override
	public final T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException {

		return readInternal(clazz, inputMessage);
	}

	@Override
	public final void write(final T t, MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException {

		final HttpHeaders headers = outputMessage.getHeaders();
		addDefaultHeaders(headers, t, contentType);

		if (outputMessage instanceof StreamingHttpOutputMessage) {
			StreamingHttpOutputMessage streamingOutputMessage = (StreamingHttpOutputMessage) outputMessage;
			streamingOutputMessage.setBody(new StreamingHttpOutputMessage.Body() {
				@Override
				public void writeTo(final OutputStream outputStream) throws IOException {
					writeInternal(t, new HttpOutputMessage() {
						@Override
						public OutputStream getBody() throws IOException {
							return outputStream;
						}
						@Override
						public HttpHeaders getHeaders() {
							return headers;
						}
					});
				}
			});
		}
		else {
			writeInternal(t, outputMessage);
			outputMessage.getBody().flush();
		}
	}

	protected void addDefaultHeaders(HttpHeaders headers, T t, MediaType contentType) throws IOException {
		if (headers.getContentType() == null) {
			MediaType contentTypeToUse = contentType;
			if (contentType == null || contentType.isWildcardType() || contentType.isWildcardSubtype()) {
				contentTypeToUse = getDefaultContentType(t);
			}
			else if (MediaType.APPLICATION_OCTET_STREAM.equals(contentType)) {
				MediaType mediaType = getDefaultContentType(t);
				contentTypeToUse = (mediaType != null ? mediaType : contentTypeToUse);
			}
			if (contentTypeToUse != null) {
				if (contentTypeToUse.getCharset() == null) {
					Charset defaultCharset = getDefaultCharset();
					if (defaultCharset != null) {
						contentTypeToUse = new MediaType(contentTypeToUse, defaultCharset);
					}
				}
				headers.setContentType(contentTypeToUse);
			}
		}
		if (headers.getContentLength() < 0 && !headers.containsKey(HttpHeaders.TRANSFER_ENCODING)) {
			Long contentLength = getContentLength(t, headers.getContentType());
			if (contentLength != null) {
				headers.setContentLength(contentLength);
			}
		}
	}

	protected MediaType getDefaultContentType(T t) throws IOException {
		List<MediaType> mediaTypes = getSupportedMediaTypes();
		return (!mediaTypes.isEmpty() ? mediaTypes.get(0) : null);
	}

	protected Long getContentLength(T t, MediaType contentType) throws IOException {
		return null;
	}

	protected abstract boolean supports(Class<?> clazz);

	protected abstract T readInternal(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;

	protected abstract void writeInternal(T t, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

}

```

#### AbstractGenericHttpMessageConverter.java

```java
package org.springframework.http.converter;
/**
 * 抽象实现
 */
public abstract class AbstractGenericHttpMessageConverter<T> extends AbstractHttpMessageConverter<T>
		implements GenericHttpMessageConverter<T> {

	protected AbstractGenericHttpMessageConverter() {
	}

	protected AbstractGenericHttpMessageConverter(MediaType supportedMediaType) {
		super(supportedMediaType);
	}

	protected AbstractGenericHttpMessageConverter(MediaType... supportedMediaTypes) {
		super(supportedMediaTypes);
	}


	@Override
	protected boolean supports(Class<?> clazz) {
		return true;
	}

	@Override
	public boolean canRead(Type type, Class<?> contextClass, MediaType mediaType) {
		return (type instanceof Class ? canRead((Class<?>) type, mediaType) : canRead(mediaType));
	}

	@Override
	public boolean canWrite(Type type, Class<?> clazz, MediaType mediaType) {
		return canWrite(clazz, mediaType);
	}

	public final void write(final T t, final Type type, MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException {

		final HttpHeaders headers = outputMessage.getHeaders();
		addDefaultHeaders(headers, t, contentType);

		if (outputMessage instanceof StreamingHttpOutputMessage) {
			StreamingHttpOutputMessage streamingOutputMessage = (StreamingHttpOutputMessage) outputMessage;
			streamingOutputMessage.setBody(new StreamingHttpOutputMessage.Body() {
				@Override
				public void writeTo(final OutputStream outputStream) throws IOException {
					writeInternal(t, type, new HttpOutputMessage() {
						@Override
						public OutputStream getBody() throws IOException {
							return outputStream;
						}
						@Override
						public HttpHeaders getHeaders() {
							return headers;
						}
					});
				}
			});
		}
		else {
			writeInternal(t, type, outputMessage);
			outputMessage.getBody().flush();
		}
	}

	@Override
	protected void writeInternal(T t, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException {

		writeInternal(t, null, outputMessage);
	}

	protected abstract void writeInternal(T t, Type type, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

}

```

##### AbstractJackson2HttpMessageConverter.java

```java
package org.springframework.http.converter.json;
/**
 * Jackson 2.6 and higher
 */
public abstract class AbstractJackson2HttpMessageConverter extends AbstractGenericHttpMessageConverter<Object> {

	public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");

	private static final MediaType TEXT_EVENT_STREAM = new MediaType("text", "event-stream");


	protected ObjectMapper objectMapper;

	private Boolean prettyPrint;

	private PrettyPrinter ssePrettyPrinter;


	protected AbstractJackson2HttpMessageConverter(ObjectMapper objectMapper) {
		init(objectMapper);
	}

	protected AbstractJackson2HttpMessageConverter(ObjectMapper objectMapper, MediaType supportedMediaType) {
		super(supportedMediaType);
		init(objectMapper);
	}

	protected AbstractJackson2HttpMessageConverter(ObjectMapper objectMapper, MediaType... supportedMediaTypes) {
		super(supportedMediaTypes);
		init(objectMapper);
	}

	protected void init(ObjectMapper objectMapper) {
		this.objectMapper = objectMapper;
		setDefaultCharset(DEFAULT_CHARSET);
		DefaultPrettyPrinter prettyPrinter = new DefaultPrettyPrinter();
		prettyPrinter.indentObjectsWith(new DefaultIndenter("  ", "\ndata:"));
		this.ssePrettyPrinter = prettyPrinter;
	}

	public void setObjectMapper(ObjectMapper objectMapper) {
		Assert.notNull(objectMapper, "ObjectMapper must not be null");
		this.objectMapper = objectMapper;
		configurePrettyPrint();
	}

	public ObjectMapper getObjectMapper() {
		return this.objectMapper;
	}

	public void setPrettyPrint(boolean prettyPrint) {
		this.prettyPrint = prettyPrint;
		configurePrettyPrint();
	}

	private void configurePrettyPrint() {
		if (this.prettyPrint != null) {
			this.objectMapper.configure(SerializationFeature.INDENT_OUTPUT, this.prettyPrint);
		}
	}


	@Override
	public boolean canRead(Class<?> clazz, MediaType mediaType) {
		return canRead(clazz, null, mediaType);
	}

	@Override
	public boolean canRead(Type type, Class<?> contextClass, MediaType mediaType) {
		if (!canRead(mediaType)) {
			return false;
		}
		JavaType javaType = getJavaType(type, contextClass);
		AtomicReference<Throwable> causeRef = new AtomicReference<Throwable>();
		if (this.objectMapper.canDeserialize(javaType, causeRef)) {
			return true;
		}
		logWarningIfNecessary(javaType, causeRef.get());
		return false;
	}

	@Override
	public boolean canWrite(Class<?> clazz, MediaType mediaType) {
		if (!canWrite(mediaType)) {
			return false;
		}
		AtomicReference<Throwable> causeRef = new AtomicReference<Throwable>();
		if (this.objectMapper.canSerialize(clazz, causeRef)) {
			return true;
		}
		logWarningIfNecessary(clazz, causeRef.get());
		return false;
	}

	protected void logWarningIfNecessary(Type type, Throwable cause) {
		if (cause == null) {
			return;
		}

		// Do not log warning for serializer not found (note: different message wording on Jackson 2.9)
		boolean debugLevel = (cause instanceof JsonMappingException &&
				(cause.getMessage().startsWith("Can not find") || cause.getMessage().startsWith("Cannot find")));

		if (debugLevel ? logger.isDebugEnabled() : logger.isWarnEnabled()) {
			String msg = "Failed to evaluate Jackson " + (type instanceof JavaType ? "de" : "") +
					"serialization for type [" + type + "]";
			if (debugLevel) {
				logger.debug(msg, cause);
			}
			else if (logger.isDebugEnabled()) {
				logger.warn(msg, cause);
			}
			else {
				logger.warn(msg + ": " + cause);
			}
		}
	}

	@Override
	protected Object readInternal(Class<?> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException {

		JavaType javaType = getJavaType(clazz, null);
		return readJavaType(javaType, inputMessage);
	}

	@Override
	public Object read(Type type, Class<?> contextClass, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException {

		JavaType javaType = getJavaType(type, contextClass);
		return readJavaType(javaType, inputMessage);
	}

	private Object readJavaType(JavaType javaType, HttpInputMessage inputMessage) {
		try {
			if (inputMessage instanceof MappingJacksonInputMessage) {
				Class<?> deserializationView = ((MappingJacksonInputMessage) inputMessage).getDeserializationView();
				if (deserializationView != null) {
					return this.objectMapper.readerWithView(deserializationView).forType(javaType).
							readValue(inputMessage.getBody());
				}
			}
			return this.objectMapper.readValue(inputMessage.getBody(), javaType);
		}
		catch (JsonProcessingException ex) {
			throw new HttpMessageNotReadableException("JSON parse error: " + ex.getOriginalMessage(), ex);
		}
		catch (IOException ex) {
			throw new HttpMessageNotReadableException("I/O error while reading input message", ex);
		}
	}

	@Override
	protected void writeInternal(Object object, Type type, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException {

		MediaType contentType = outputMessage.getHeaders().getContentType();
		JsonEncoding encoding = getJsonEncoding(contentType);
		JsonGenerator generator = this.objectMapper.getFactory().createGenerator(outputMessage.getBody(), encoding);
		try {
			writePrefix(generator, object);

			Object value = object;
			Class<?> serializationView = null;
			FilterProvider filters = null;
			JavaType javaType = null;

			if (object instanceof MappingJacksonValue) {
				MappingJacksonValue container = (MappingJacksonValue) object;
				value = container.getValue();
				serializationView = container.getSerializationView();
				filters = container.getFilters();
			}
			if (type != null && value != null && TypeUtils.isAssignable(type, value.getClass())) {
				javaType = getJavaType(type, null);
			}

			ObjectWriter objectWriter = (serializationView != null ?
					this.objectMapper.writerWithView(serializationView) : this.objectMapper.writer());
			if (filters != null) {
				objectWriter = objectWriter.with(filters);
			}
			if (javaType != null && javaType.isContainerType()) {
				objectWriter = objectWriter.forType(javaType);
			}
			SerializationConfig config = objectWriter.getConfig();
			if (contentType != null && contentType.isCompatibleWith(TEXT_EVENT_STREAM) &&
					config.isEnabled(SerializationFeature.INDENT_OUTPUT)) {
				objectWriter = objectWriter.with(this.ssePrettyPrinter);
			}
			objectWriter.writeValue(generator, value);

			writeSuffix(generator, object);
			generator.flush();
		}
		catch (JsonProcessingException ex) {
			throw new HttpMessageNotWritableException("Could not write JSON: " + ex.getOriginalMessage(), ex);
		}
	}

	protected void writePrefix(JsonGenerator generator, Object object) throws IOException {
	}

	protected void writeSuffix(JsonGenerator generator, Object object) throws IOException {
	}

	protected JavaType getJavaType(Type type, Class<?> contextClass) {
		TypeFactory typeFactory = this.objectMapper.getTypeFactory();
		if (contextClass != null) {
			ResolvableType resolvedType = ResolvableType.forType(type);
			if (type instanceof TypeVariable) {
				ResolvableType resolvedTypeVariable = resolveVariable(
						(TypeVariable<?>) type, ResolvableType.forClass(contextClass));
				if (resolvedTypeVariable != ResolvableType.NONE) {
					return typeFactory.constructType(resolvedTypeVariable.resolve());
				}
			}
			else if (type instanceof ParameterizedType && resolvedType.hasUnresolvableGenerics()) {
				ParameterizedType parameterizedType = (ParameterizedType) type;
				Class<?>[] generics = new Class<?>[parameterizedType.getActualTypeArguments().length];
				Type[] typeArguments = parameterizedType.getActualTypeArguments();
				for (int i = 0; i < typeArguments.length; i++) {
					Type typeArgument = typeArguments[i];
					if (typeArgument instanceof TypeVariable) {
						ResolvableType resolvedTypeArgument = resolveVariable(
								(TypeVariable<?>) typeArgument, ResolvableType.forClass(contextClass));
						if (resolvedTypeArgument != ResolvableType.NONE) {
							generics[i] = resolvedTypeArgument.resolve();
						}
						else {
							generics[i] = ResolvableType.forType(typeArgument).resolve();
						}
					}
					else {
						generics[i] = ResolvableType.forType(typeArgument).resolve();
					}
				}
				return typeFactory.constructType(ResolvableType.
						forClassWithGenerics(resolvedType.getRawClass(), generics).getType());
			}
		}
		return typeFactory.constructType(type);
	}

	private ResolvableType resolveVariable(TypeVariable<?> typeVariable, ResolvableType contextType) {
		ResolvableType resolvedType;
		if (contextType.hasGenerics()) {
			resolvedType = ResolvableType.forType(typeVariable, contextType);
			if (resolvedType.resolve() != null) {
				return resolvedType;
			}
		}

		ResolvableType superType = contextType.getSuperType();
		if (superType != ResolvableType.NONE) {
			resolvedType = resolveVariable(typeVariable, superType);
			if (resolvedType.resolve() != null) {
				return resolvedType;
			}
		}
		for (ResolvableType ifc : contextType.getInterfaces()) {
			resolvedType = resolveVariable(typeVariable, ifc);
			if (resolvedType.resolve() != null) {
				return resolvedType;
			}
		}
		return ResolvableType.NONE;
	}

	protected JsonEncoding getJsonEncoding(MediaType contentType) {
		if (contentType != null && contentType.getCharset() != null) {
			Charset charset = contentType.getCharset();
			for (JsonEncoding encoding : JsonEncoding.values()) {
				if (charset.name().equals(encoding.getJavaName())) {
					return encoding;
				}
			}
		}
		return JsonEncoding.UTF8;
	}

	@Override
	protected MediaType getDefaultContentType(Object object) throws IOException {
		if (object instanceof MappingJacksonValue) {
			object = ((MappingJacksonValue) object).getValue();
		}
		return super.getDefaultContentType(object);
	}

	@Override
	protected Long getContentLength(Object object, MediaType contentType) throws IOException {
		if (object instanceof MappingJacksonValue) {
			object = ((MappingJacksonValue) object).getValue();
		}
		return super.getContentLength(object, contentType);
	}

}

```

###### MappingJackson2HttpMessageConverter.java

```java
package org.springframework.http.converter.json;


/**
 * Jackson转换Json
 */
public class MappingJackson2HttpMessageConverter extends AbstractJackson2HttpMessageConverter {

	private String jsonPrefix;

	public MappingJackson2HttpMessageConverter() {
		this(Jackson2ObjectMapperBuilder.json().build());
	}

	public MappingJackson2HttpMessageConverter(ObjectMapper objectMapper) {
		super(objectMapper, MediaType.APPLICATION_JSON, new MediaType("application", "*+json"));
	}

	public void setJsonPrefix(String jsonPrefix) {
		this.jsonPrefix = jsonPrefix;
	}

	public void setPrefixJson(boolean prefixJson) {
		this.jsonPrefix = (prefixJson ? ")]}', " : null);
	}


	@Override
	@SuppressWarnings("deprecation")
	protected void writePrefix(JsonGenerator generator, Object object) throws IOException {
		if (this.jsonPrefix != null) {
			generator.writeRaw(this.jsonPrefix);
		}
		String jsonpFunction =
				(object instanceof MappingJacksonValue ? ((MappingJacksonValue) object).getJsonpFunction() : null);
		if (jsonpFunction != null) {
			generator.writeRaw("/**/");
			generator.writeRaw(jsonpFunction + "(");
		}
	}

	@Override
	@SuppressWarnings("deprecation")
	protected void writeSuffix(JsonGenerator generator, Object object) throws IOException {
		String jsonpFunction =
				(object instanceof MappingJacksonValue ? ((MappingJacksonValue) object).getJsonpFunction() : null);
		if (jsonpFunction != null) {
			generator.writeRaw(");");
		}
	}

}

```

###### MappingJackson2XmlHttpMessageConverter.java

```java
package org.springframework.http.converter.xml;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.dataformat.xml.XmlMapper;

import org.springframework.http.MediaType;
import org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter;
import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;
import org.springframework.util.Assert;

/**
 * Jackson转换XML
 */
public class MappingJackson2XmlHttpMessageConverter extends AbstractJackson2HttpMessageConverter {

	public MappingJackson2XmlHttpMessageConverter() {
		this(Jackson2ObjectMapperBuilder.xml().build());
	}

	public MappingJackson2XmlHttpMessageConverter(ObjectMapper objectMapper) {
		super(objectMapper, new MediaType("application", "xml"),
				new MediaType("text", "xml"),
				new MediaType("application", "*+xml"));
		Assert.isInstanceOf(XmlMapper.class, objectMapper, "XmlMapper required");
	}

	@Override
	public void setObjectMapper(ObjectMapper objectMapper) {
		Assert.isInstanceOf(XmlMapper.class, objectMapper, "XmlMapper required");
		super.setObjectMapper(objectMapper);
	}

}

```

# Advice

## RequestBodyAdvice.java

```java
package org.springframework.web.servlet.mvc.method.annotation;
/**
 * 允许在读取和转换为对象之前定制请求，还允许在将结果对象作为@RequestBody或HttpEntity方法参数传递到控制器方法之前对其进行处理。
 */
public interface RequestBodyAdvice {

	boolean supports(MethodParameter methodParameter, Type targetType,
			Class<? extends HttpMessageConverter<?>> converterType);

	Object handleEmptyBody(Object body, HttpInputMessage inputMessage, MethodParameter parameter,
			Type targetType, Class<? extends HttpMessageConverter<?>> converterType);

	HttpInputMessage beforeBodyRead(HttpInputMessage inputMessage, MethodParameter parameter,
			Type targetType, Class<? extends HttpMessageConverter<?>> converterType) throws IOException;

	Object afterBodyRead(Object body, HttpInputMessage inputMessage, MethodParameter parameter,
			Type targetType, Class<? extends HttpMessageConverter<?>> converterType);

}

```

### RequestBodyAdviceAdapter.java

```java
package org.springframework.web.servlet.mvc.method.annotation;
/**
 * 适配抽象实现
 */
public abstract class RequestBodyAdviceAdapter implements RequestBodyAdvice {

	@Override
	public Object handleEmptyBody(Object body, HttpInputMessage inputMessage,
			MethodParameter parameter, Type targetType,
			Class<? extends HttpMessageConverter<?>> converterType) {

		return body;
	}

	@Override
	public HttpInputMessage beforeBodyRead(HttpInputMessage inputMessage, MethodParameter parameter,
			Type targetType, Class<? extends HttpMessageConverter<?>> converterType)
			throws IOException {

		return inputMessage;
	}

	@Override
	public Object afterBodyRead(Object body, HttpInputMessage inputMessage, MethodParameter parameter,
			Type targetType, Class<? extends HttpMessageConverter<?>> converterType) {

		return body;
	}

}

```

#### JsonViewRequestBodyAdvice.java

```java
package org.springframework.web.servlet.mvc.method.annotation;
/**
 * 处理Jackson的json视图注解
 */
public class JsonViewRequestBodyAdvice extends RequestBodyAdviceAdapter {

	@Override
	public boolean supports(MethodParameter methodParameter, Type targetType,
			Class<? extends HttpMessageConverter<?>> converterType) {

		return (AbstractJackson2HttpMessageConverter.class.isAssignableFrom(converterType) &&
				methodParameter.getParameterAnnotation(JsonView.class) != null);
	}

	@Override
	public HttpInputMessage beforeBodyRead(HttpInputMessage inputMessage, MethodParameter methodParameter,
			Type targetType, Class<? extends HttpMessageConverter<?>> selectedConverterType) throws IOException {

		JsonView annotation = methodParameter.getParameterAnnotation(JsonView.class);
		Class<?>[] classes = annotation.value();
		if (classes.length != 1) {
			throw new IllegalArgumentException(
					"@JsonView only supported for request body advice with exactly 1 class argument: " + methodParameter);
		}
		return new MappingJacksonInputMessage(inputMessage.getBody(), inputMessage.getHeaders(), classes[0]);
	}

}

```

## ResponseBodyAdvice.java

```java
package org.springframework.web.servlet.mvc.method.annotation;

import org.springframework.core.MethodParameter;
import org.springframework.http.MediaType;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;

/**
 * 允许在@ResponseBody或ResponseEntity控制器方法执行之后，但在使用HttpMessageConverter编写主体之前自定义响应
 */
public interface ResponseBodyAdvice<T> {

	boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType);

	T beforeBodyWrite(T body, MethodParameter returnType, MediaType selectedContentType,
			Class<? extends HttpMessageConverter<?>> selectedConverterType,
			ServerHttpRequest request, ServerHttpResponse response);

}

```

### AbstractMappingJacksonResponseBodyAdvice.java

```java
package org.springframework.web.servlet.mvc.method.annotation;
/**
 * Jackson抽象实现
 */
public abstract class AbstractMappingJacksonResponseBodyAdvice implements ResponseBodyAdvice<Object> {

	@Override
	public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
		return AbstractJackson2HttpMessageConverter.class.isAssignableFrom(converterType);
	}

	@Override
	public final Object beforeBodyWrite(Object body, MethodParameter returnType,
			MediaType contentType, Class<? extends HttpMessageConverter<?>> converterType,
			ServerHttpRequest request, ServerHttpResponse response) {

		MappingJacksonValue container = getOrCreateContainer(body);
		beforeBodyWriteInternal(container, contentType, returnType, request, response);
		return container;
	}

	protected MappingJacksonValue getOrCreateContainer(Object body) {
		return (body instanceof MappingJacksonValue ? (MappingJacksonValue) body : new MappingJacksonValue(body));
	}

	protected abstract void beforeBodyWriteInternal(MappingJacksonValue bodyContainer, MediaType contentType,
			MethodParameter returnType, ServerHttpRequest request, ServerHttpResponse response);

}

```

#### JsonViewResponseBodyAdvice.java

```java
package org.springframework.web.servlet.mvc.method.annotation;
/**
 * Jackson的Json视图支持
 */
public class JsonViewResponseBodyAdvice extends AbstractMappingJacksonResponseBodyAdvice {

	@Override
	public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
		return super.supports(returnType, converterType) && returnType.hasMethodAnnotation(JsonView.class);
	}

	@Override
	protected void beforeBodyWriteInternal(MappingJacksonValue bodyContainer, MediaType contentType,
			MethodParameter returnType, ServerHttpRequest request, ServerHttpResponse response) {

		JsonView annotation = returnType.getMethodAnnotation(JsonView.class);
		Class<?>[] classes = annotation.value();
		if (classes.length != 1) {
			throw new IllegalArgumentException(
					"@JsonView only supported for response body advice with exactly 1 class argument: " + returnType);
		}
		bodyContainer.setSerializationView(classes[0]);
	}

}

```

### FastJsonViewResponseBodyAdvice.java

```java
package com.alibaba.fastjson.support.spring;
/**
 * fastjson的json视图支持
 */
@Order
@ControllerAdvice
public class FastJsonViewResponseBodyAdvice implements ResponseBodyAdvice<Object> {

    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
        return FastJsonHttpMessageConverter.class.isAssignableFrom(converterType) && returnType.hasMethodAnnotation(FastJsonView.class);
    }

    public FastJsonContainer beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        FastJsonContainer container = getOrCreateContainer(body);
        beforeBodyWriteInternal(container, selectedContentType, returnType, request, response);
        return container;
    }

    private FastJsonContainer getOrCreateContainer(Object body) {
        return (body instanceof FastJsonContainer ? (FastJsonContainer) body : new FastJsonContainer(body));

    }

    protected void beforeBodyWriteInternal(FastJsonContainer container, MediaType contentType,
                                           MethodParameter returnType, ServerHttpRequest request, ServerHttpResponse response) {
        FastJsonView annotation = returnType.getMethodAnnotation(FastJsonView.class);

        FastJsonFilter[] include = annotation.include();
        FastJsonFilter[] exclude = annotation.exclude();
        PropertyPreFilters filters = new PropertyPreFilters();
        for (FastJsonFilter item : include) {
            filters.addFilter(item.clazz(),item.props());
        }
        for (FastJsonFilter item : exclude) {
            filters.addFilter(item.clazz()).addExcludes(item.props());
        }
        container.setFilters(filters);
    }
}

```

### JSONPResponseBodyAdvice.java

```java
package com.alibaba.fastjson.support.spring;
/**
 * fastjson的jsonp支持
 */
@Order(Integer.MIN_VALUE)//before FastJsonViewResponseBodyAdvice
@ControllerAdvice
public class JSONPResponseBodyAdvice implements ResponseBodyAdvice<Object> {

    public final Log logger = LogFactory.getLog(this.getClass());

    public JSONPResponseBodyAdvice() {
    }


    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {


        return FastJsonHttpMessageConverter.class.isAssignableFrom(converterType)
                &&
                (returnType.getContainingClass().isAnnotationPresent(ResponseJSONP.class) || returnType.hasMethodAnnotation(ResponseJSONP.class));
    }

    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType,
                                  Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request,
                                  ServerHttpResponse response) {

        ResponseJSONP responseJsonp = returnType.getMethodAnnotation(ResponseJSONP.class);
        if(responseJsonp == null){
            responseJsonp = returnType.getContainingClass().getAnnotation(ResponseJSONP.class);
        }

        HttpServletRequest servletRequest = ((ServletServerHttpRequest) request).getServletRequest();
        String callbackMethodName = servletRequest.getParameter(responseJsonp.callback());

        if (!IOUtils.isValidJsonpQueryParam(callbackMethodName)) {
            if (logger.isDebugEnabled()) {
                logger.debug("Invalid jsonp parameter value:" + callbackMethodName);
            }
            callbackMethodName = null;
        }

        JSONPObject jsonpObject = new JSONPObject(callbackMethodName);
        jsonpObject.addParameter(body);
        beforeBodyWriteInternal(jsonpObject, selectedContentType, returnType, request, response);
        return jsonpObject;
    }


    public void beforeBodyWriteInternal(JSONPObject jsonpObject, MediaType contentType,
                                        MethodParameter returnType, ServerHttpRequest request, ServerHttpResponse response) {
        //MediaType contentTypeToUse = getContentType(contentType, request, response);
        //response.getHeaders().setContentType(contentTypeToUse);
    }

    protected MediaType getContentType(MediaType contentType, ServerHttpRequest request, ServerHttpResponse response) {
        return FastJsonHttpMessageConverter.APPLICATION_JAVASCRIPT;
    }
}

```

### ControllerAdviceBean.java

```java
package org.springframework.web.method;
/**
 * 封装被@ControllerAdvice注解的bean
 */
public class ControllerAdviceBean implements Ordered {

	private final Object bean;

	private final BeanFactory beanFactory;

	private final int order;

	private final Set<String> basePackages;

	private final List<Class<?>> assignableTypes;

	private final List<Class<? extends Annotation>> annotations;


	public ControllerAdviceBean(Object bean) {
		this(bean, null);
	}

	public ControllerAdviceBean(String beanName, BeanFactory beanFactory) {
		this((Object) beanName, beanFactory);
	}

	private ControllerAdviceBean(Object bean, BeanFactory beanFactory) {
		this.bean = bean;
		this.beanFactory = beanFactory;
		Class<?> beanType;

		if (bean instanceof String) {
			String beanName = (String) bean;
			Assert.hasText(beanName, "Bean name must not be null");
			Assert.notNull(beanFactory, "BeanFactory must not be null");
			if (!beanFactory.containsBean(beanName)) {
				throw new IllegalArgumentException("BeanFactory [" + beanFactory +
						"] does not contain specified controller advice bean '" + beanName + "'");
			}
			beanType = this.beanFactory.getType(beanName);
			this.order = initOrderFromBeanType(beanType);
		}
		else {
			Assert.notNull(bean, "Bean must not be null");
			beanType = bean.getClass();
			this.order = initOrderFromBean(bean);
		}

		ControllerAdvice annotation =
				AnnotatedElementUtils.findMergedAnnotation(beanType, ControllerAdvice.class);

		if (annotation != null) {
			this.basePackages = initBasePackages(annotation);
			this.assignableTypes = Arrays.asList(annotation.assignableTypes());
			this.annotations = Arrays.asList(annotation.annotations());
		}
		else {
			this.basePackages = Collections.emptySet();
			this.assignableTypes = Collections.emptyList();
			this.annotations = Collections.emptyList();
		}
	}

	@Override
	public int getOrder() {
		return this.order;
	}

	public Class<?> getBeanType() {
		Class<?> clazz = (this.bean instanceof String ?
				this.beanFactory.getType((String) this.bean) : this.bean.getClass());
		return ClassUtils.getUserClass(clazz);
	}

	public Object resolveBean() {
		return (this.bean instanceof String ? this.beanFactory.getBean((String) this.bean) : this.bean);
	}

	public boolean isApplicableToBeanType(Class<?> beanType) {
		if (!hasSelectors()) {
			return true;
		}
		else if (beanType != null) {
			for (String basePackage : this.basePackages) {
				if (beanType.getName().startsWith(basePackage)) {
					return true;
				}
			}
			for (Class<?> clazz : this.assignableTypes) {
				if (ClassUtils.isAssignable(clazz, beanType)) {
					return true;
				}
			}
			for (Class<? extends Annotation> annotationClass : this.annotations) {
				if (AnnotationUtils.findAnnotation(beanType, annotationClass) != null) {
					return true;
				}
			}
		}
		return false;
	}

	private boolean hasSelectors() {
		return (!this.basePackages.isEmpty() || !this.assignableTypes.isEmpty() || !this.annotations.isEmpty());
	}


	@Override
	public boolean equals(Object other) {
		if (this == other) {
			return true;
		}
		if (!(other instanceof ControllerAdviceBean)) {
			return false;
		}
		ControllerAdviceBean otherAdvice = (ControllerAdviceBean) other;
		return (this.bean.equals(otherAdvice.bean) && this.beanFactory == otherAdvice.beanFactory);
	}

	@Override
	public int hashCode() {
		return this.bean.hashCode();
	}

	@Override
	public String toString() {
		return this.bean.toString();
	}

	public static List<ControllerAdviceBean> findAnnotatedBeans(ApplicationContext applicationContext) {
		List<ControllerAdviceBean> beans = new ArrayList<ControllerAdviceBean>();
		for (String name : BeanFactoryUtils.beanNamesForTypeIncludingAncestors(applicationContext, Object.class)) {
			if (applicationContext.findAnnotationOnBean(name, ControllerAdvice.class) != null) {
				beans.add(new ControllerAdviceBean(name, applicationContext));
			}
		}
		return beans;
	}

	private static int initOrderFromBean(Object bean) {
		return (bean instanceof Ordered ? ((Ordered) bean).getOrder() : initOrderFromBeanType(bean.getClass()));
	}

	private static int initOrderFromBeanType(Class<?> beanType) {
		return OrderUtils.getOrder(beanType, Ordered.LOWEST_PRECEDENCE);
	}

	private static Set<String> initBasePackages(ControllerAdvice annotation) {
		Set<String> basePackages = new LinkedHashSet<String>();
		for (String basePackage : annotation.basePackages()) {
			if (StringUtils.hasText(basePackage)) {
				basePackages.add(adaptBasePackage(basePackage));
			}
		}
		for (Class<?> markerClass : annotation.basePackageClasses()) {
			basePackages.add(adaptBasePackage(ClassUtils.getPackageName(markerClass)));
		}
		return basePackages;
	}

	private static String adaptBasePackage(String basePackage) {
		return (basePackage.endsWith(".") ? basePackage : basePackage + ".");
	}

}

```



### RequestResponseBodyAdviceChain.java

```java
package org.springframework.web.servlet.mvc.method.annotation;
/**
 * 请求响应体通知链
 */
class RequestResponseBodyAdviceChain implements RequestBodyAdvice, ResponseBodyAdvice<Object> {

	private final List<Object> requestBodyAdvice = new ArrayList<Object>(4);

	private final List<Object> responseBodyAdvice = new ArrayList<Object>(4);


	/**
	 * Create an instance from a list of objects that are either of type
	 * {@code ControllerAdviceBean} or {@code RequestBodyAdvice}.
	 */
	public RequestResponseBodyAdviceChain(List<Object> requestResponseBodyAdvice) {
		initAdvice(requestResponseBodyAdvice);
	}

	private void initAdvice(List<Object> requestResponseBodyAdvice) {
		if (requestResponseBodyAdvice == null) {
			return;
		}
		for (Object advice : requestResponseBodyAdvice) {
			Class<?> beanType = (advice instanceof ControllerAdviceBean ?
					((ControllerAdviceBean) advice).getBeanType() : advice.getClass());
			if (RequestBodyAdvice.class.isAssignableFrom(beanType)) {
				this.requestBodyAdvice.add(advice);
			}
			else if (ResponseBodyAdvice.class.isAssignableFrom(beanType)) {
				this.responseBodyAdvice.add(advice);
			}
		}
	}

	private List<Object> getAdvice(Class<?> adviceType) {
		if (RequestBodyAdvice.class == adviceType) {
			return this.requestBodyAdvice;
		}
		else if (ResponseBodyAdvice.class == adviceType) {
			return this.responseBodyAdvice;
		}
		else {
			throw new IllegalArgumentException("Unexpected adviceType: " + adviceType);
		}
	}


	@Override
	public boolean supports(MethodParameter param, Type type, Class<? extends HttpMessageConverter<?>> converterType) {
		throw new UnsupportedOperationException("Not implemented");
	}

	@Override
	public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
		throw new UnsupportedOperationException("Not implemented");
	}

	@Override
	public Object handleEmptyBody(Object body, HttpInputMessage inputMessage, MethodParameter parameter,
			Type targetType, Class<? extends HttpMessageConverter<?>> converterType) {

		for (RequestBodyAdvice advice : getMatchingAdvice(parameter, RequestBodyAdvice.class)) {
			if (advice.supports(parameter, targetType, converterType)) {
				body = advice.handleEmptyBody(body, inputMessage, parameter, targetType, converterType);
			}
		}
		return body;
	}

	@Override
	public HttpInputMessage beforeBodyRead(HttpInputMessage request, MethodParameter parameter,
			Type targetType, Class<? extends HttpMessageConverter<?>> converterType) throws IOException {

		for (RequestBodyAdvice advice : getMatchingAdvice(parameter, RequestBodyAdvice.class)) {
			if (advice.supports(parameter, targetType, converterType)) {
				request = advice.beforeBodyRead(request, parameter, targetType, converterType);
			}
		}
		return request;
	}

	@Override
	public Object afterBodyRead(Object body, HttpInputMessage inputMessage, MethodParameter parameter,
			Type targetType, Class<? extends HttpMessageConverter<?>> converterType) {

		for (RequestBodyAdvice advice : getMatchingAdvice(parameter, RequestBodyAdvice.class)) {
			if (advice.supports(parameter, targetType, converterType)) {
				body = advice.afterBodyRead(body, inputMessage, parameter, targetType, converterType);
			}
		}
		return body;
	}

	@Override
	public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType contentType,
			Class<? extends HttpMessageConverter<?>> converterType,
			ServerHttpRequest request, ServerHttpResponse response) {

		return processBody(body, returnType, contentType, converterType, request, response);
	}

	@SuppressWarnings("unchecked")
	private <T> Object processBody(Object body, MethodParameter returnType, MediaType contentType,
			Class<? extends HttpMessageConverter<?>> converterType,
			ServerHttpRequest request, ServerHttpResponse response) {

		for (ResponseBodyAdvice<?> advice : getMatchingAdvice(returnType, ResponseBodyAdvice.class)) {
			if (advice.supports(returnType, converterType)) {
				body = ((ResponseBodyAdvice<T>) advice).beforeBodyWrite((T) body, returnType,
						contentType, converterType, request, response);
			}
		}
		return body;
	}

	@SuppressWarnings("unchecked")
	private <A> List<A> getMatchingAdvice(MethodParameter parameter, Class<? extends A> adviceType) {
		List<Object> availableAdvice = getAdvice(adviceType);
		if (CollectionUtils.isEmpty(availableAdvice)) {
			return Collections.emptyList();
		}
		List<A> result = new ArrayList<A>(availableAdvice.size());
		for (Object advice : availableAdvice) {
			if (advice instanceof ControllerAdviceBean) {
				ControllerAdviceBean adviceBean = (ControllerAdviceBean) advice;
				if (!adviceBean.isApplicableToBeanType(parameter.getContainingClass())) {
					continue;
				}
				advice = adviceBean.resolveBean();
			}
			if (adviceType.isAssignableFrom(advice.getClass())) {
				result.add((A) advice);
			}
		}
		return result;
	}

}

```



# HandlerMethodArgumentResolver

## RequstMappingHandlerAdapter.java

```java
	private List<HandlerMethodArgumentResolver> getDefaultArgumentResolvers() {
		List<HandlerMethodArgumentResolver> resolvers = new ArrayList<HandlerMethodArgumentResolver>();

		// Annotation-based argument resolution
		resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), false));
		resolvers.add(new RequestParamMapMethodArgumentResolver());
		resolvers.add(new PathVariableMethodArgumentResolver());
		resolvers.add(new PathVariableMapMethodArgumentResolver());
		resolvers.add(new MatrixVariableMethodArgumentResolver());
		resolvers.add(new MatrixVariableMapMethodArgumentResolver());
		resolvers.add(new ServletModelAttributeMethodProcessor(false));
		resolvers.add(new RequestResponseBodyMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
		resolvers.add(new RequestPartMethodArgumentResolver(getMessageConverters(), this.requestResponseBodyAdvice));
		resolvers.add(new RequestHeaderMethodArgumentResolver(getBeanFactory()));
		resolvers.add(new RequestHeaderMapMethodArgumentResolver());
		resolvers.add(new ServletCookieValueMethodArgumentResolver(getBeanFactory()));
		resolvers.add(new ExpressionValueMethodArgumentResolver(getBeanFactory()));
		resolvers.add(new SessionAttributeMethodArgumentResolver());
		resolvers.add(new RequestAttributeMethodArgumentResolver());

		// Type-based argument resolution
		resolvers.add(new ServletRequestMethodArgumentResolver());
		resolvers.add(new ServletResponseMethodArgumentResolver());
		resolvers.add(new HttpEntityMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
		resolvers.add(new RedirectAttributesMethodArgumentResolver());
		resolvers.add(new ModelMethodProcessor());
		resolvers.add(new MapMethodProcessor());
		resolvers.add(new ErrorsMethodArgumentResolver());
		resolvers.add(new SessionStatusMethodArgumentResolver());
		resolvers.add(new UriComponentsBuilderMethodArgumentResolver());

		// Custom arguments
		if (getCustomArgumentResolvers() != null) {
			resolvers.addAll(getCustomArgumentResolvers());
		}

		// Catch-all
		resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), true));
		resolvers.add(new ServletModelAttributeMethodProcessor(true));

		return resolvers;
	}
```

## HandlerMethodArgumentResolver.java

```java
package org.springframework.web.method.support;
/**
 * 处理器方法参数解析器
 */
public interface HandlerMethodArgumentResolver {

	boolean supportsParameter(MethodParameter parameter);

	Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception;

}

```



### AbstractNamedValueMethodArgumentResolver.java

```java
package org.springframework.web.method.annotation;

/**
 * 支持${}和#{}，支持WebDataBinder，支持命名参数，支持缓存
 */
public abstract class AbstractNamedValueMethodArgumentResolver implements HandlerMethodArgumentResolver {

	private final ConfigurableBeanFactory configurableBeanFactory;
	//用来解析${}和#{}
	private final BeanExpressionContext expressionContext;

	private final Map<MethodParameter, NamedValueInfo> namedValueInfoCache =
			new ConcurrentHashMap<MethodParameter, NamedValueInfo>(256);


	public AbstractNamedValueMethodArgumentResolver() {
		this.configurableBeanFactory = null;
		this.expressionContext = null;
	}

	public AbstractNamedValueMethodArgumentResolver(ConfigurableBeanFactory beanFactory) {
		this.configurableBeanFactory = beanFactory;
		this.expressionContext =
				(beanFactory != null ? new BeanExpressionContext(beanFactory, new RequestScope()) : null);
	}


	@Override
	public final Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {

		NamedValueInfo namedValueInfo = getNamedValueInfo(parameter);
		MethodParameter nestedParameter = parameter.nestedIfOptional();

		Object resolvedName = resolveStringValue(namedValueInfo.name);
		if (resolvedName == null) {
			throw new IllegalArgumentException(
					"Specified name must not resolve to null: [" + namedValueInfo.name + "]");
		}

		Object arg = resolveName(resolvedName.toString(), nestedParameter, webRequest);
		if (arg == null) {
			if (namedValueInfo.defaultValue != null) {
				arg = resolveStringValue(namedValueInfo.defaultValue);
			}
			else if (namedValueInfo.required && !nestedParameter.isOptional()) {
				handleMissingValue(namedValueInfo.name, nestedParameter, webRequest);
			}
			arg = handleNullValue(namedValueInfo.name, arg, nestedParameter.getNestedParameterType());
		}
		else if ("".equals(arg) && namedValueInfo.defaultValue != null) {
			arg = resolveStringValue(namedValueInfo.defaultValue);
		}

		if (binderFactory != null) {
			WebDataBinder binder = binderFactory.createBinder(webRequest, null, namedValueInfo.name);
			try {
				arg = binder.convertIfNecessary(arg, parameter.getParameterType(), parameter);
			}
			catch (ConversionNotSupportedException ex) {
				throw new MethodArgumentConversionNotSupportedException(arg, ex.getRequiredType(),
						namedValueInfo.name, parameter, ex.getCause());
			}
			catch (TypeMismatchException ex) {
				throw new MethodArgumentTypeMismatchException(arg, ex.getRequiredType(),
						namedValueInfo.name, parameter, ex.getCause());

			}
		}

		handleResolvedValue(arg, namedValueInfo.name, parameter, mavContainer, webRequest);

		return arg;
	}

	private NamedValueInfo getNamedValueInfo(MethodParameter parameter) {
		NamedValueInfo namedValueInfo = this.namedValueInfoCache.get(parameter);
		if (namedValueInfo == null) {
			namedValueInfo = createNamedValueInfo(parameter);
			namedValueInfo = updateNamedValueInfo(parameter, namedValueInfo);
			this.namedValueInfoCache.put(parameter, namedValueInfo);
		}
		return namedValueInfo;
	}

	protected abstract NamedValueInfo createNamedValueInfo(MethodParameter parameter);

	private NamedValueInfo updateNamedValueInfo(MethodParameter parameter, NamedValueInfo info) {
		String name = info.name;
		if (info.name.isEmpty()) {
			name = parameter.getParameterName();
			if (name == null) {
				throw new IllegalArgumentException(
						"Name for argument type [" + parameter.getNestedParameterType().getName() +
						"] not available, and parameter name information not found in class file either.");
			}
		}
		String defaultValue = (ValueConstants.DEFAULT_NONE.equals(info.defaultValue) ? null : info.defaultValue);
		return new NamedValueInfo(name, info.required, defaultValue);
	}

	private Object resolveStringValue(String value) {
		if (this.configurableBeanFactory == null) {
			return value;
		}
		String placeholdersResolved = this.configurableBeanFactory.resolveEmbeddedValue(value);
		BeanExpressionResolver exprResolver = this.configurableBeanFactory.getBeanExpressionResolver();
		if (exprResolver == null) {
			return value;
		}
		return exprResolver.evaluate(placeholdersResolved, this.expressionContext);
	}

	protected abstract Object resolveName(String name, MethodParameter parameter, NativeWebRequest request)
			throws Exception;

	protected void handleMissingValue(String name, MethodParameter parameter, NativeWebRequest request)
			throws Exception {

		handleMissingValue(name, parameter);
	}

	protected void handleMissingValue(String name, MethodParameter parameter) throws ServletException {
		throw new ServletRequestBindingException("Missing argument '" + name +
				"' for method parameter of type " + parameter.getNestedParameterType().getSimpleName());
	}

	private Object handleNullValue(String name, Object value, Class<?> paramType) {
		if (value == null) {
			if (Boolean.TYPE.equals(paramType)) {
				return Boolean.FALSE;
			}
			else if (paramType.isPrimitive()) {
				throw new IllegalStateException("Optional " + paramType.getSimpleName() + " parameter '" + name +
						"' is present but cannot be translated into a null value due to being declared as a " +
						"primitive type. Consider declaring it as object wrapper for the corresponding primitive type.");
			}
		}
		return value;
	}

	protected void handleResolvedValue(Object arg, String name, MethodParameter parameter,
			ModelAndViewContainer mavContainer, NativeWebRequest webRequest) {
	}

	protected static class NamedValueInfo {

		private final String name;

		private final boolean required;

		private final String defaultValue;

		public NamedValueInfo(String name, boolean required, String defaultValue) {
			this.name = name;
			this.required = required;
			this.defaultValue = defaultValue;
		}
	}

}
```

#### ExpressionValueMethodArgumentResolver.java

```java
package org.springframework.web.method.annotation;

import javax.servlet.ServletException;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.core.MethodParameter;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.context.request.NativeWebRequest;

/**
 * 解析@Value中的值,支持${}和#{}，解析后的结果当成参数值
 */
public class ExpressionValueMethodArgumentResolver extends AbstractNamedValueMethodArgumentResolver {

	public ExpressionValueMethodArgumentResolver(ConfigurableBeanFactory beanFactory) {
		super(beanFactory);
	}


	@Override
	public boolean supportsParameter(MethodParameter parameter) {
		return parameter.hasParameterAnnotation(Value.class);
	}

	@Override
	protected NamedValueInfo createNamedValueInfo(MethodParameter parameter) {
		Value annotation = parameter.getParameterAnnotation(Value.class);
		return new ExpressionValueNamedValueInfo(annotation);
	}

	@Override
	protected Object resolveName(String name, MethodParameter parameter, NativeWebRequest webRequest) throws Exception {
		// No name to resolve
		return null;
	}

	@Override
	protected void handleMissingValue(String name, MethodParameter parameter) throws ServletException {
		throw new UnsupportedOperationException("@Value is never required: " + parameter.getMethod());
	}


	private static class ExpressionValueNamedValueInfo extends NamedValueInfo {

		private ExpressionValueNamedValueInfo(Value annotation) {
			super("@Value", false, annotation.value());
		}
	}

}

```

### AbstractMessageConverterMethodArgumentResolver.java

```java
package org.springframework.web.servlet.mvc.method.annotation;
/**
 * 使用HttpMessageConverter读取请求体内容
 */
public abstract class AbstractMessageConverterMethodArgumentResolver implements HandlerMethodArgumentResolver {

	private static final Set<HttpMethod> SUPPORTED_METHODS =
			EnumSet.of(HttpMethod.POST, HttpMethod.PUT, HttpMethod.PATCH);

	private static final Object NO_VALUE = new Object();


	protected final Log logger = LogFactory.getLog(getClass());

	protected final List<HttpMessageConverter<?>> messageConverters;

	protected final List<MediaType> allSupportedMediaTypes;

	private final RequestResponseBodyAdviceChain advice;

	public AbstractMessageConverterMethodArgumentResolver(List<HttpMessageConverter<?>> converters) {
		this(converters, null);
	}

	public AbstractMessageConverterMethodArgumentResolver(List<HttpMessageConverter<?>> converters,
			List<Object> requestResponseBodyAdvice) {

		Assert.notEmpty(converters, "'messageConverters' must not be empty");
		this.messageConverters = converters;
		this.allSupportedMediaTypes = getAllSupportedMediaTypes(converters);
		this.advice = new RequestResponseBodyAdviceChain(requestResponseBodyAdvice);
	}

	private static List<MediaType> getAllSupportedMediaTypes(List<HttpMessageConverter<?>> messageConverters) {
		Set<MediaType> allSupportedMediaTypes = new LinkedHashSet<MediaType>();
		for (HttpMessageConverter<?> messageConverter : messageConverters) {
			allSupportedMediaTypes.addAll(messageConverter.getSupportedMediaTypes());
		}
		List<MediaType> result = new ArrayList<MediaType>(allSupportedMediaTypes);
		MediaType.sortBySpecificity(result);
		return Collections.unmodifiableList(result);
	}

	protected RequestResponseBodyAdviceChain getAdvice() {
		return this.advice;
	}


	protected <T> Object readWithMessageConverters(NativeWebRequest webRequest, MethodParameter parameter,
			Type paramType) throws IOException, HttpMediaTypeNotSupportedException, HttpMessageNotReadableException {

		HttpInputMessage inputMessage = createInputMessage(webRequest);
		return readWithMessageConverters(inputMessage, parameter, paramType);
	}

	@SuppressWarnings("unchecked")
	protected <T> Object readWithMessageConverters(HttpInputMessage inputMessage, MethodParameter parameter,
			Type targetType) throws IOException, HttpMediaTypeNotSupportedException, HttpMessageNotReadableException {

		MediaType contentType;
		boolean noContentType = false;
		try {
			contentType = inputMessage.getHeaders().getContentType();
		}
		catch (InvalidMediaTypeException ex) {
			throw new HttpMediaTypeNotSupportedException(ex.getMessage());
		}
		if (contentType == null) {
			noContentType = true;
			contentType = MediaType.APPLICATION_OCTET_STREAM;
		}

		Class<?> contextClass = (parameter != null ? parameter.getContainingClass() : null);
		Class<T> targetClass = (targetType instanceof Class ? (Class<T>) targetType : null);
		if (targetClass == null) {
			ResolvableType resolvableType = (parameter != null ?
					ResolvableType.forMethodParameter(parameter) : ResolvableType.forType(targetType));
			targetClass = (Class<T>) resolvableType.resolve();
		}

		HttpMethod httpMethod = ((HttpRequest) inputMessage).getMethod();
		Object body = NO_VALUE;

		try {
			inputMessage = new EmptyBodyCheckingHttpInputMessage(inputMessage);

			for (HttpMessageConverter<?> converter : this.messageConverters) {
				Class<HttpMessageConverter<?>> converterType = (Class<HttpMessageConverter<?>>) converter.getClass();
				if (converter instanceof GenericHttpMessageConverter) {
					GenericHttpMessageConverter<?> genericConverter = (GenericHttpMessageConverter<?>) converter;
					if (genericConverter.canRead(targetType, contextClass, contentType)) {
						if (logger.isDebugEnabled()) {
							logger.debug("Read [" + targetType + "] as \"" + contentType + "\" with [" + converter + "]");
						}
						if (inputMessage.getBody() != null) {
							inputMessage = getAdvice().beforeBodyRead(inputMessage, parameter, targetType, converterType);
							body = genericConverter.read(targetType, contextClass, inputMessage);
							body = getAdvice().afterBodyRead(body, inputMessage, parameter, targetType, converterType);
						}
						else {
							body = getAdvice().handleEmptyBody(null, inputMessage, parameter, targetType, converterType);
						}
						break;
					}
				}
				else if (targetClass != null) {
					if (converter.canRead(targetClass, contentType)) {
						if (logger.isDebugEnabled()) {
							logger.debug("Read [" + targetType + "] as \"" + contentType + "\" with [" + converter + "]");
						}
						if (inputMessage.getBody() != null) {
							inputMessage = getAdvice().beforeBodyRead(inputMessage, parameter, targetType, converterType);
							body = ((HttpMessageConverter<T>) converter).read(targetClass, inputMessage);
							body = getAdvice().afterBodyRead(body, inputMessage, parameter, targetType, converterType);
						}
						else {
							body = getAdvice().handleEmptyBody(null, inputMessage, parameter, targetType, converterType);
						}
						break;
					}
				}
			}
		}
		catch (IOException ex) {
			throw new HttpMessageNotReadableException("I/O error while reading input message", ex);
		}

		if (body == NO_VALUE) {
			if (httpMethod == null || !SUPPORTED_METHODS.contains(httpMethod) ||
					(noContentType && inputMessage.getBody() == null)) {
				return null;
			}
			throw new HttpMediaTypeNotSupportedException(contentType, this.allSupportedMediaTypes);
		}

		return body;
	}

	protected ServletServerHttpRequest createInputMessage(NativeWebRequest webRequest) {
		HttpServletRequest servletRequest = webRequest.getNativeRequest(HttpServletRequest.class);
		return new ServletServerHttpRequest(servletRequest);
	}

	protected void validateIfApplicable(WebDataBinder binder, MethodParameter parameter) {
		Annotation[] annotations = parameter.getParameterAnnotations();
		for (Annotation ann : annotations) {
			Validated validatedAnn = AnnotationUtils.getAnnotation(ann, Validated.class);
			if (validatedAnn != null || ann.annotationType().getSimpleName().startsWith("Valid")) {
				Object hints = (validatedAnn != null ? validatedAnn.value() : AnnotationUtils.getValue(ann));
				Object[] validationHints = (hints instanceof Object[] ? (Object[]) hints : new Object[] {hints});
				binder.validate(validationHints);
				break;
			}
		}
	}

	protected boolean isBindExceptionRequired(WebDataBinder binder, MethodParameter parameter) {
		int i = parameter.getParameterIndex();
		Class<?>[] paramTypes = parameter.getMethod().getParameterTypes();
		boolean hasBindingResult = (paramTypes.length > (i + 1) && Errors.class.isAssignableFrom(paramTypes[i + 1]));
		return !hasBindingResult;
	}

	protected Object adaptArgumentIfNecessary(Object arg, MethodParameter parameter) {
		return (parameter.isOptional() ? OptionalResolver.resolveValue(arg) : arg);
	}


	private static class EmptyBodyCheckingHttpInputMessage implements HttpInputMessage {

		private final HttpHeaders headers;

		private final InputStream body;

		private final HttpMethod method;


		public EmptyBodyCheckingHttpInputMessage(HttpInputMessage inputMessage) throws IOException {
			this.headers = inputMessage.getHeaders();
			InputStream inputStream = inputMessage.getBody();
			if (inputStream == null) {
				this.body = null;
			}
			else if (inputStream.markSupported()) {
				inputStream.mark(1);
				this.body = (inputStream.read() != -1 ? inputStream : null);
				inputStream.reset();
			}
			else {
				PushbackInputStream pushbackInputStream = new PushbackInputStream(inputStream);
				int b = pushbackInputStream.read();
				if (b == -1) {
					this.body = null;
				}
				else {
					this.body = pushbackInputStream;
					pushbackInputStream.unread(b);
				}
			}
			this.method = ((HttpRequest) inputMessage).getMethod();
		}

		@Override
		public HttpHeaders getHeaders() {
			return this.headers;
		}

		@Override
		public InputStream getBody() throws IOException {
			return this.body;
		}

		public HttpMethod getMethod() {
			return this.method;
		}
	}

	@UsesJava8
	private static class OptionalResolver {

		public static Object resolveValue(Object value) {
			if (value == null || (value instanceof Collection && ((Collection) value).isEmpty()) ||
					(value instanceof Object[] && ((Object[]) value).length == 0)) {
				return Optional.empty();
			}
			return Optional.of(value);
		}
	}

}

```



# END