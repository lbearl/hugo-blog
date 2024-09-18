---
title: "Making EF work with DotNet SPA 2.0.0 prerelease"
date: "2018-02-21"
categories: 
  - "software-development"
---

The Microsoft.DotNet.Web.Spa.ProjectTemplates::2.0.0-rc2-final SPA template currently has a little issue with making EF work properly. If you encounter this issue you'll notice that running `Update-Database` in the Package Manager Console results in `ng serve` being run, which prevents the migrations from being applied.

To fix the issue, we need to slightly change `Program.cs` and `Startup.cs` as follows:

```
    public class Program
    {
        public static void Main(string[] args)
        {
            // This needs to have all of the code originally in BuildWebHost
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>().Build().Run();
        }

        // This is only invoked by EF
        public static IWebHost BuildWebHost(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((ctx, cfg) =>
            {
                cfg.SetBasePath(Directory.GetCurrentDirectory())
                  .AddJsonFile("appSettings.json", true) // require the appsettings file!
                  .AddEnvironmentVariables();
            })
                .UseStartup<Startup>().UseSetting("DesignTime", "true").Build();
    }

```

In `Startup.cs` look for `app.UseSpa` inside of `Configure`:

```
            app.UseSpa(spa =>
            {
                // To learn more about options for serving an Angular SPA from ASP.NET Core,
                // see https://go.microsoft.com/fwlink/?linkid=864501

                spa.Options.SourcePath = "ClientApp";
                // Get the "DesignTime" configuration param set above
                bool.TryParse(Configuration["DesignTime"], out var designTime);
                // Only launch the server if we are in development mode AND we are not in design time (which is EF)
                if (env.IsDevelopment() && !designTime)
                {
                    spa.UseAngularCliServer(npmScript: "start");
                }
            });
```

With those changes Entity Framework starts working perfectly again. Hope this saves people a few hours.
