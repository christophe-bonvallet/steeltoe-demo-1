# Start

install cf cli and connect to pivotal
```sh
cf login -a https://api.run.pivotal.io/ -u christophe.bonvallet@photoweb.fr -p 7y4kr432fNPTBaJ@
```

Create your project 
```sh
dotnet new webapi --name Formation.WeatherForecast.Service

cd Formation.WeatherForecast.Service

# for check, publish works !
dotnet publish

# add steeltoe package
dotnet add package Steeltoe.common.hosting
```

Update file Program.cs
```c#
// add 
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                })
                .UseCloudHosting(9339); // <-- new line;

// we are now on port 9339
```

```sh
# Add new extensions
dotnet add package Steeltoe.extensions.configuration.cloudfoundrycore
```

Update file Program.cs again
```c#
// add 
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                })
                .UseCloudHosting(9339)
                .AddCloudFoundry();// <-- new line;

// We are plugued to cloudFoundry
```

```sh
cf push Formation.WeatherForecast.Service -p ./bin/Debug/netcoreapp3.1/publish --random-route # random-route is here for dev purpose
```


```sh
# Add package for dynamic loging (change loging dynamicly without reload)
dotnet add package Steeltoe.extensions.logging.dynamiclogger
```


```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "steeltoe": "information" // Add this line
    }
  },
  "AllowedHosts": "*"
}
```


```sh
# Add package for dynamic loging (change loging dynamicly without reload)
dotnet add package Steeltoe.management.cloudFoundryCore
```

in startup.cs, add in ConfigureServices 
```c#
            services.AddCloudFoundryActuators(Configuration);
            services.AddRefreshActuator(Configuration);
```

in startup.cs, add in Configure 
```c#
            app.UseCloudFoundryActuators();
            app.UseRefreshActuator();
```

add node management in appsettings
````json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "steeltoe": "information"
    }
  },
  "management": { // Node added
    "endpoints": {
      "enabled": true,
      "path": "/cloudfoundryapplication", 
    }
  },
  "AllowedHosts": "*"
}
```

now Health appear