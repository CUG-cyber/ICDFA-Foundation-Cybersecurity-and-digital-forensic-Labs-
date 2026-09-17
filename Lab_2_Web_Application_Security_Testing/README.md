## 🌐 Lab 2 - Web Application Security Testing

**Objective:** Move from network reconnaissance to application-layer testing against DVWA and Mutillidae, focusing on request/response inspection, file-upload validation weaknesses, and SQL injection.

**Tools used:** Browser DevTools, Burp Suite Community Edition, Nikto, DIRB, manual SQL injection, SQLMap (metadata enumeration only)

**Key activities performed**
- Inspected raw HTTP requests/responses and session cookies via DevTools and Burp
- Tested DVWA's file-upload feature with harmless files, comparing extension handling vs. client-supplied `Content-Type` spoofing
- Confirmed uploaded files are stored in a web-accessible, executable directory
- Ran Nikto against the target and analysed 5 key configuration findings (version disclosure, `phpinfo.php` exposure, HTTP TRACE/XST risk, directory indexing, exposed phpMyAdmin)
- Ran DIRB for content/path discovery and correlated results with the upload-storage risk
- Mapped user-controlled input parameters in Mutillidae
- Performed manual SQL injection testing (`'`, `' OR '1'='1`, `' AND '1'='2`) and explained the resulting query logic
- Verified findings using SQLMap, restricted strictly to database/table metadata enumeration (no data exfiltration or modification)
- Documented a defensive-controls table mapping each weak practice to its recommended fix

**Highlights from findings**
- File upload accepts spoofed `Content-Type` headers, enabling MIME-based bypass of client-side validation
- Boolean-based SQL injection confirmed manually in Mutillidae's login form and cross-verified with SQLMap (MySQL back end, 7 databases enumerated)
- Multiple unauthenticated, web-accessible administrative and diagnostic endpoints identified (`/phpMyAdmin/`, `/phpinfo.php`, `/dav/`, `/twiki/`)

📄 Full evidence, request/response tables, Nikto/DIRB output, and defensive recommendations: see `Report`

---

