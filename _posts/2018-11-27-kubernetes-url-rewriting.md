---
  title: Kubernetes Ingress URL Rewriting.
  header:
    subheading: Rewrite your Ingress URLs to match your application
---

# Relative URLS
When using relative URLS behind an ingress, you may find that you have issues if your service is served at 
some site like `[my.domain]/[external-service-path]/` Navigating relative to the domain will miss out the `/[external-service-path]/` 
part of the URL.

Furthermore, your app is probably expecting to be served at the root `/`, so you'll need to strip out the `/[external-service-path]/` 
out of requests to your application.

You can use the following annotations to achieve this:

```yaml
ingress:
  [...]
  annotations: 
    nginx.ingress.kubernetes.io/add-base-url: "true"
    nginx.ingress.kubernetes.io/base-url-scheme: "https"
    nginx.ingress.kubernetes.io/rewrite-target: "/"
```

The `add-base-url` annotation tells NGINX to prepend a `<base href="..." />` tag to the `head` section of your HTML which 
will be used as a base for relative links.

`base-url-scheme` is used to, for example, specify https. Really you should be serving everything over https, so this _ought_ 
to be the default! What is perhaps surprising is that there is a separate setting `force-ssl-redirect`. If you are forcing
the page to be served over HTTPS then you will surely need that as the `base-url-scheme`!

`rewrite-target: "/"` is used when your application is expecting to be served at `/`, i.e. your application "base URL" does not match
the one you have set up in your Ingress: requests to `[my.domain]/[external-service-path]/[relative-path]` will be redirected to 
`[internal-ip]/[relative-path]`
