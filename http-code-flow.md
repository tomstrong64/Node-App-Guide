# HTTP Request Code Flow Standards

## Flow
```text
1. Does the route exist? → No → 404 Not Found
2. Does the route have a resource? → No → pass to controller layer
3. Is the user logged in? → No → 401 Unauthorized
4. Does the resource exist? → No → 404 Not Found
5. Does the user have access to the resource? → No → 404 Not Found
6. Does the user have access to the route? → No → 403 Forbidden
7. Is the request content valid? → No → 400 Bad Request
8. Otherwise → proceed to controller, handle other status codes as required
```

---

## Rules
- Always return the **most restrictive applicable status code early**.  
- Prefer **404 for hidden resources** (don’t expose existence).  
- Distinguish between:
  - `401 Unauthorized` → not logged in.  
  - `403 Forbidden` → logged in but not allowed.  
  - `400 Bad Request` → validation errors or malformed input.  

---

## Checklist
- [ ] Route check first (`404` if not found).  
- [ ] Auth before resource checks (`401` vs `403`).  
- [ ] Hide existence where needed → use `404` instead of `403`.  
- [ ] Validate body/content before controller.  
- [ ] Controller only handles actual logic and other applicable codes.  

---

## References
- [HTTP Status Codes (MDN)](https://developer.mozilla.org/docs/Web/HTTP/Status)  
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
