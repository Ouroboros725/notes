https://stackoverflow.com/questions/53873153/should-health-checks-call-other-app-health-checks

Service A is ready if it can serve business requests. So if being able to reach B is part of what it *needs* to do (which it seems it is) then it should check B.

An advantage of having A check for B is you can then [fail fast on a bad rolling upgrade][2]. Say your A gets misconfigured so that the upgrade features a wrong connection detail for B - maybe B's service name is injected as an environment variable and the new version has a typo. If your A instances check to Bs on startup then you can more easily ensure that the upgrade fails and that no traffic goes to the new misconfigured Pods. For more on this see https://medium.com/spire-labs/utilizing-kubernetes-liveness-and-readiness-probes-to-automatically-recover-from-failure-2fe0314f2b2e

It would typically be enough for A to check B's liveness endpoint or any minimal availability endpoint rather than B's readiness endpoint. This is because kubernetes will be [checking B's readiness probe for you anyway][1] so any B instance that A can reach will be a ready one. Calling B's liveness endpoint rather than readiness can make a difference if B's [readiness endpoint performs more checks than the liveness one][3]. Keep in mind that kubernetes will be calling these probes regularly - [readiness as well as liveness - they both have a period][4]. The difference is whether the Pod is withdrawn from serving traffic (if readiness fails) or restarted (if liveness fails). You're not trying to do [end-to-end transaction checks][5], you want these checks to contain minimal logic and not use up too much load.

It is preferable if the code within A's implementation of readiness does the check rather than doing the check at the k8s level (in the Pod spec itself). It is second-best to do it at the k8s level as ideally you want to know that the code running in the container really does connect.

Another way to check dependent services are available [is with a check in an initContainer][6]. Using initContainers avoids seeing multiple restarts during startup (by ensuring correct ordering) but doing the checks to dependencies through probes can go deeper (if implemented in the app's code) and the probes will continue to run periodically after startup. So it can be advantageous to use both.

Be careful of checking other services from readiness too liberally as it can lead to cascading unavailability. For example, if a backend briefly goes down and a frontend is probing to it then the frontend will also become unavailable and so won't be able to display a good error message. You might want to start with simple probes and carefully add complexity as you go.

[1]: https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-setting-up-health-checks-with-readiness-and-liveness-probes
[2]: https://blog.sebastian-daschner.com/entries/zero-downtime-updates-kubernetes
[3]: https://medium.com/metrosystemsro/kubernetes-readiness-liveliness-probes-best-practices-86c3cd9f0b4a
[4]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes
[5]: https://dzone.com/articles/monitoring-microservices-with-health-checks
[6]: https://medium.com/@xcoulon/initializing-containers-in-order-with-kubernetes-18173b9cc222

---

Referencing Microsoft's [Implementing Resilient Applications](https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/implement-resilient-applications/) tutorials. Specifically the [Health monitoring](https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/implement-resilient-applications/monitor-app-health), it is suggested that if the overall status of the current service is dependent on the status of a dependency then the healthy status of the service should only be healthy if its dependencies are healthy

>However, the MVC web application of eShopOnContainers has multiple dependencies on the rest of the microservices. Therefore, it calls one `AddUrlCheck` method for each microservice, as shown in the following example:


    // Startup.cs from the MVC web app
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();
            services.Configure<AppSettings>(Configuration);
            services.AddHealthChecks(checks =>
            {
                checks.AddUrlCheck(Configuration["CatalogUrl"]);
                checks.AddUrlCheck(Configuration["OrderingUrl"]);
                checks.AddUrlCheck(Configuration["BasketUrl"]);
                checks.AddUrlCheck(Configuration["IdentityUrl"]);
            });
        }
    }

>**Thus, a microservice will not provide a “healthy” status until all its checks are healthy as well.**

***emphasis mine***

So to more directly answer your question about

>Should the readiness health check for A make a call to the readiness health check for API B because of the dependency?

I would say yes it should. Especially if the health of the dependency `B` directly affects the stability of `A` .