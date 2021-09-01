https://stackoverflow.com/questions/39935190/contentcachingresponsewrapper-produces-empty-response

After couple of hours of struggling, I've finally found the solution.

In short,  `ContentCachingResponseWrapper.copyBodyToResponse()`  should be called in the end of the filter method.

`ContentCachingResponseWrapper`  caches the response body by reading it from response output stream. So, the stream becomes empty. To write response back to the output stream  `ContentCachingResponseWrapper.copyBodyToResponse()`  should be used.

---

http://laiyijie.me/2017/07/13/spring-accesslog-filter

```java
public class AccessLogFilter extends OncePerRequestFilter {
    private static final Logger logger = LogManager.getLogger(AccessLogFilter.class);

    private String usernameKey = "username";
    private Integer payloadMaxLength = 1024;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        Long startTime = System.currentTimeMillis();
        ContentCachingRequestWrapper requestWrapper = new ContentCachingRequestWrapper(request);
        ContentCachingResponseWrapper responseWrapper = new ContentCachingResponseWrapper(response);

        filterChain.doFilter(requestWrapper, responseWrapper);

        String requestPayload = getPayLoad(requestWrapper.getContentAsByteArray(),
                request.getCharacterEncoding());
        String responsePayload = getPayLoad(responseWrapper.getContentAsByteArray(),
                response.getCharacterEncoding());
        responseWrapper.copyBodyToResponse();
        BLog.accessJsonLogBuilder()
            .addRequestPayLoad(requestPayload)
            .addResponsePayLoad(responsePayload)
            .put(request, usernameKey)
            .put(response)
            .put("_COST_", System.currentTimeMillis() - startTime)
            .log();
    }

    private String getPayLoad(byte[] buf, String characterEncoding) {
        String payload = "";
        if (buf == null) return payload;
        if (buf.length > 0) {
            int length = Math.min(buf.length, getPayloadMaxLength());
            try {
                payload = new String(buf, 0, length, characterEncoding);
            } catch (UnsupportedEncodingException ex) {
                payload = "[unknown]";
            }
        }
        return payload;
    }


    public String getUsernameKey() {
        return usernameKey;
    }

    public void setUsernameKey(String usernameKey) {
        this.usernameKey = usernameKey;
    }

    public Integer getPayloadMaxLength() {
        return payloadMaxLength;
    }

    public void setPayloadMaxLength(Integer payloadMaxLength) {
        this.payloadMaxLength = payloadMaxLength;
    }
}
```