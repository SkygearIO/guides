# Cross-Origin policy

For web platform, browsers has a built-in security mechanism "Cross-Origin Resource Sharing". It prevents unauthorized access to resources from another domain.

By default, Skygear only allow access to your app in the same domain. It would allow micro-services deployed to Skygear, since they shares the domain. However, if you would like to access your app's resources from another domain, you need to configure Cross-Origin policy in **Advanced Settings** page of Portal.

### References

* [Documentation on CORS mechanism](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

