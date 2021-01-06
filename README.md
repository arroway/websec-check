# websec-check

repo and tooling for [wikimo checklist](https://wiki.mozilla.org/Security/FirefoxOperations#Security_Checklist)

Markdown issue template:

```markdown
Risk Management
---------------
* [ ] The service must have performed a Rapid Risk Assessment
* [ ] The service must be registered via a [New Service issue](https://github.com/mozilla-services/foxsec/issues/new?template=01_NewService.md&labels=New%20Service&assignee=psiinon&title=New%20Service:%20). You need to have access to the mozilla-services GitHub organization to be able to view and create this issue (ping secops if you don't have access).

Infrastructure
--------------

* [ ] Access and application logs must be archived for a minimum of 90 days
* [ ] Use [Modern](https://wiki.mozilla.org/Security/Server_Side_TLS#Modern_compatibility) or [Intermediate](https://wiki.mozilla.org/Security/Server_Side_TLS#Intermediate_compatibility) TLS
* [ ] Set HSTS to 31536000 (1 year)
  * `strict-transport-security: max-age=31536000`
  * [ ] If the service is not hosted under `services.mozilla.com`, it must be manually added to [Firefox's preloaded pins](https://dxr.mozilla.org/mozilla-central/source/security/manager/tools/PreloadedHPKPins.json#184). This only applies to production services, not short-lived experiments.
* [ ] Correctly set client IP
  * [ ] Confirm client ip is in the proper location in [X-Forwarded-For](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For), modifying what is sent from the client if needed. AWS and GCP's load balancers will do this automatically.
  * [ ] Make sure the web server and the application get the true client IP by configuring trusted IP's within Nginx or Apache
    * Nginx: [ngx_http_realip_module](https://nginx.org/en/docs/http/ngx_http_realip_module.html)
    * Apache: [mod_remoteip](https://httpd.apache.org/docs/2.4/mod/mod_remoteip.html)
  * [ ] If you have a service-oriented architecture, you must always be able to find the IP of the client that sent the initial request. We recommend passing along the `X-Forwarded-For` to all back-end services.
* If service has an admin panels, it must:
  * [ ] only be available behind Mozilla VPN (which provides MFA)
  * [ ] require Auth0 authentication
  * [ ] Enforce a CSP with `frame-ancestors 'none'` to prevent iframe related attacks, such as [DOM-Based CSRF](https://www.youtube.com/watch?v=Femsrx0m9bU)
* [ ] Build and deploy main or -slim variants of official language-specific base docker images e.g. [node](https://hub.docker.com/_/node/), [python](https://hub.docker.com/_/python/), or [rust](https://hub.docker.com/_/rust/) and contact secops@ if you want to use other variants

Development
-----------
* [ ] Ensure your code repository is configured and located appropriately:
  * [ ] Application built internally should be hosted in trusted GitHub organizations (mozilla, mozilla-services, mozilla-bteam, mozilla-conduit, mozilla-mobile, taskcluster). Sometimes we build and deploy applications we don't fully control. In those cases, the Dockerfile that builds the application container should be hosted in its own repository in a trusted organization.
  * [ ] Secure your repository by implementing [Mozilla's GitHub security standard](https://github.com/mozilla-services/GitHub-Audit/blob/master/docs/checklist.md).
* [ ] Sign all release tags, and ideally commits as well
  * Developers should [configure git to sign all tags](http://micropipes.com/blog//2016/08/31/signing-your-commits-on-github-with-a-gpg-key/) and upload their PGP fingerprint to https://login.mozilla.com
  * The signature verification will eventually become a requirement to shipping a release to staging & prod: the tag being deployed in the pipeline must have a matching tag in git signed by a project owner. This control is designed to reduce the risk of a 3rd party GitHub integration from compromising our source code.
* [ ] Enable [automated security fix PRs on GitHub](https://help.github.com/en/articles/configuring-automated-security-fixes#managing-automated-security-fixes-for-your-repository) for vulnerabilities in 3rd-party dependencies
* [ ] Keep 3rd-party libraries up to date (in addition to the security updates)
  * For NodeJS applications, use [dependabot](https://dependabot.com/), [renovate](https://renovateapp.com/), or [GreenKeeper](https://greenkeeper.io/)
  * For Python, use ``pip list --outdated`` or [requires.io](https://requires.io/) or pyup outdated checks
  * For Rust, use `cargo update` and [cargo upgrade](https://github.com/killercup/cargo-edit#cargo-upgrade) when changing versions
* [ ] Integrate static code analysis in CI, and avoid merging code with issues
    * Javascript applications should use ESLint with the [Mozilla ruleset](https://developer.mozilla.org/en-US/docs/ESLint)
    * Python applications should use [Bandit](https://github.com/openstack/bandit)
    * Go applications should use the [Go Meta Linter](https://github.com/alecthomas/gometalinter)
    * Use whitelisting mechanisms in these tools to deal with false positives

Dual Sign Off
------------
* [ ] Services that push data to Firefox clients must require a dual sign off on every change, implemented in their admin panels
  * This mechanism must be reviewed and approved by the Firefox Operations Security team before being enabled in production

Logging
-------
* [ ] Publish detailed logs in [mozlog](https://github.com/mozilla-services/Dockerflow/blob/master/docs/mozlog.md) format (**APP-MOZLOG**)
  * Business logic must be logged with app specific codes (see [FxA](https://github.com/mozilla/fxa-auth-server/blob/master/docs/api.md#defined-errors))
  * Access control failures must be logged at WARN level

Web Applications
----------------
* [ ] Must have a CSP with
  * [ ] a report-uri pointing to the service's own `/__cspreport__` endpoint
  * [ ] web API responses should return `default-src 'none'; frame-ancestors 'none'; base-uri 'none'; report-uri /__cspreport__` to disallowing all content rendering, framing, and report violations
  * [ ] if default-src is not `none`, frame-src, and object-src should be `none` or only allow specific origins
  * [ ] no use of unsafe-inline or unsafe-eval in script-src, style-src, and img-src
* [ ] Third-party javascript must be pinned to specific versions using [Subresource Integrity (SRI)](https://infosec.mozilla.org/guidelines/web_security#subresource-integrity)
* [ ] Web APIs:
  * [ ] must set a non-HTML content-type on all responses, including 300s, 400s and 500s
  * [ ] Web APIs should export an OpenAPI (Swagger) to facilitate automated vulnerability tests
  * [ ] Should use authentication tokens with a unique pattern which is easily parsed with a regexp. This should allow inclusion into a token scanning service in the future. (E.g. prefix `mgp-` + 20 hex digits would match the regexp `\bmgp-[0-9A-Fa-f]{20}\b`)
* [ ] for [Cookies](https://wiki.mozilla.org/Security/Guidelines/Web_Security#Cookies):
  * [ ] Set the Secure and HTTPOnly flags
  * [ ] Use a sensible Expiration
  * [ ] Use the prefix `__Host-` for the cookie name
* [ ] Make sure your application gets an A+ on the [Mozilla Observatory](https://observatory.mozilla.org/)
* [ ] Confirm your application doesn't fail the [ZAP Security Baseline](https://github.com/zaproxy/zaproxy/wiki/ZAP-Baseline-Scan):
  1. Register your service as described in the Risk Management section (if you are not registering your service you can run the scan as described at: https://github.com/zaproxy/zaproxy/wiki/ZAP-Baseline-Scan)
  2. Go to [The STMO baseline dashboard](https://sql.telemetry.mozilla.org/dashboard/security-baseline-top-level-scores) (if you do not have access to the dashboard, contact secops@)
  3. Filter for your service name and check that the "failures" column is 0. Click on each individual site for additional information about any failures.
  4. If you need to document exceptions to the baseline e.g. to mark a search form as CSRF exempt, contact secops@ or ping 'psiinon' on github to get the scan config updated

Security Features
-----------------
* [ ] Authentication of end-users should be via FxA. Authentication of Mozillians should be via Auth0/SSO. Any exceptions must be approved by the security team.
* [ ] Session Management should be via existing and well regarded frameworks. In all cases you should contact the security team for a design and implementation review
  * Store session keys server side (typically in a db) so that they can be revoked immediately.
  * Session keys must be changed on login to prevent session fixation attacks.
  * Session cookies must have HttpOnly and Secure flags set and the SameSite attribute set to 'strict' or 'lax' (which allows external regular links to login).
  * For more information about potential pitfalls see the [OWASP Session Management Cheat Sheet](https://www.owasp.org/index.php/Session_Management_Cheat_Sheet)
* [ ] When using cookies for session management, make sure you have CSRF protections in place, which in 99% of cases is [SameSite cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#SameSite_cookies). If you can't use SameSite, use anti CSRF tokens. There are two exceptions to implementing CSRF protection:
  * Forms that don't change state (e.g. search forms) don't need CSRF protection and can indicate that by setting the 'data-no-csrf' form attribute (this tells our ZAP scanner to ignore those forms when testing for CSRF).
  * Sites that don't use cookies for anything sensitive can ignore CSRF protection. A lot of modern sites prefer to use local-storage JWTs for session management, which aren't vulnerable to CSRF (but must have a rock solid CSP).
* [ ] Access Control should be via existing and well regarded frameworks. If you really do need to roll your own then contact the security team for a design and implementation review.
* [ ] If you are building a core Firefox service, consider adding it to the list of restricted domains in the preference `extensions.webextensions.restrictedDomains`. This will prevent a malicious extension from being able to steal sensitive information from it, see [bug 1415644](https://bugzilla.mozilla.org/show_bug.cgi?id=1415644).

Databases
---------
* [ ] All SQL queries must be parameterized, not concatenated
* [ ] Applications must use accounts with limited GRANTS when connecting to databases
  * In particular, applications **must not use admin or owner accounts**, to decrease the impact of a sql injection vulnerability.

Common issues
-------------
* [ ] User data must be [escaped for the right context](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet#XSS_Prevention_Rules_Summary) prior to reflecting it
  * When inserting user generated html into an html context:
    * Python applications should use [Bleach](https://github.com/mozilla/bleach)
    * Javascript applications should use [DOMPurify](https://github.com/cure53/DOMPurify/)
* [ ] Apply sensible limits to user inputs, see [input validation](https://wiki.mozilla.org/WebAppSec/Secure_Coding_Guidelines#Input_Validation)
  * POST body size should be small (<500kB) unless explicitly needed
* [ ] When allowing users to upload or generate content, make sure to host that content on a separate domain (eg. firefoxusercontent.com, etc.). This will prevent malicious content from having access to storage and cookies from the origin.
  * Also use this technique to host rich content you can't protect with a CSP, such as metrics reports, wiki pages, etc.
* [ ] When managing permissions, make sure access controls are enforced server-side
* [ ] If an authenticated user accesses protected resource, make sure the pages with those resource arent cached and served up to unauthenticated users (like via a CDN).
* [ ] If handling cryptographic keys, must have a mechanism to handle quarterly key rotations
  * Keys used to sign sessions don't need a rotation mechanism if destroying all sessions is acceptable in case of emergency.
* [ ] Do not proxy requests from users without strong limitations and filtering (see [Pocket UserData vulnerability](https://www.gnu.gl/blog/Posts/multiple-vulnerabilities-in-pocket/)). Don't proxy requests to [link local, loopback, or private networks](https://en.wikipedia.org/wiki/Reserved_IP_addresses#IPv4) or DNS that resolves to addresses in those ranges (i.e. 169.254.0.0/16, 127.0.0.0/8, 10.0.0.0/8, 100.64.0.0/10, 172.16.0.0/12, 192.168.0.0/16, 198.18.0.0/15).
* [ ] Do not use `target="_blank"` in external links unless you also use `rel="noopener noreferrer"` (to prevent [Reverse Tabnabbing](https://www.owasp.org/index.php/Reverse_Tabnabbing))
```
