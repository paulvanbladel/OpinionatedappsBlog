+++
date = "2015-02-24T14:08:40+00:00"
draft = false
title = "WebApi AspNet.Vnext Mono with Entity Framework via docker on Ubuntu Linux (part 1/n)"
slug = "webapi-aspnet-vnext-mono-via-docker-on-ubuntu-linux-part-1"

+++

## Introduction
I have been working lately quite insensively in the Linux world. Time again for some .Net.
Let's see what happened in the mean time with AspNet Vnext and let's explore a Katana based web api.
But we will not completely leave the Linux world. We'll use Mono and run the web api on an ubuntu server and use Docker. We'll connect the web api to a database backend via Entity Framework 6. Indeed, quite a lot of new technologies. Let's start with the web api in visual studio.

## A NodeJS inspired AspNet Vnext
It was definitely time for a decent successor for System.Web. It's very clear that AspNet Vnext got some inspiration in the middleware-based approach of NodeJs. 
I'll spend not that much time here on the web api because there is plenty of information on the net on this. I'll save my energy for the more interesting Mono and Docker adventures when hosting the web api on an ubuntu server.

In NodeJs a middleware component is injected via app.use(), we'll do something similar in .Net now. In the Startup class we'll do the following:

```language-csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Owin;
using System.Web.Http;
namespace OwinMonoDocker
{
    public class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            var webApiConfiguration = ConfigureWebApi();
            app.UseWebApi(webApiConfiguration);
        }
        
        private HttpConfiguration ConfigureWebApi()
        {
            var config = new HttpConfiguration();
            config.MapHttpAttributeRoutes();
            config.Routes.MapHttpRoute(
                "DefaultApi",
                "api/{controller}/{id}",
                new { id = RouteParameter.Optional });
            return config;
        }
    }
}

```
Now, we will self-host our web api, via following code in the main method of the console application
```language-csharp
using Microsoft.Owin.Hosting;
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace OwinMonoDocker
{
    class Program
    {
        static void Main(string[] args)
        {
            string baseUri = "http://*:8080";
            // in debug in visual studio you will need 
            //string baseUri = "http://localhost:8080";

            Console.WriteLine("Starting web Server...");
            WebApp.Start<Startup>(baseUri);
            Console.WriteLine("Server running at {0} - press Enter to quit. ", baseUri);
            Console.WriteLine("I'm running on {0} directly from assembly {1}", Environment.OSVersion, System.Reflection.Assembly.GetEntryAssembly().FullName);
            Console.ReadLine();
        }
    }
}
```
As you can see were printing the OS Version when running this web api. In a later post in this series, you will be surprised when UNIX pops up over there :)

```
Starting web Server...
Server running at http://*:8080 - press Enter to quit.
I'm running on Unix 3.13.0.40 directly from assembly OwinMonoDocker, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
```

While in the visual studio debug environment on a windows box you'll see of course:

```
Starting web Server...
Server running at http://localhost:8080 - press Enter to quit.
I'm running on Microsoft Windows NT 6.2.9200.0 directly from assembly OwinMonoDocker, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
```
You could also use Monodevelop on a linux box and then you'll never see again "Microsoft Windows" on your screen. I'm fine with visual studio on windows, for the time being.

Our web api will manage Customers:
```language-csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace OwinMonoDocker
{
    public class Customer
    {
        public int Id { get; set; }
        public string Name { get; set; }
    }
}
```
Finally we'll need our customer controller:
```language-csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Web.Http;
using System.Net.Http;
using OwinMonoDocker;
using System.Data.Entity;

namespace OwinMonoDocker
{
    public class CustomersController : ApiController
    {
        ApplicationDbContext _Db = new ApplicationDbContext();

        [HttpGet]
        [Route("api/Customers/MimicCustomerImport/{numberOfRecords}")]
        public async Task<IHttpActionResult> MimicCustomerImport(int numberOfRecords)
        {
            for (int i = 0; i < numberOfRecords; i++)
            {
                _Db.Customers.Add(new Customer { Name = DateTime.Now.ToShortTimeString() + " " + i });    
            }
            await _Db.SaveChangesAsync();
            return Ok();
        }

        
        public IEnumerable<Customer> Get()
        {
            return _Db.Customers;
        }


        public async Task<Customer> Get(int id)
        {
            var customer = await _Db.Customers.FirstOrDefaultAsync(c => c.Id == id);
            if (customer == null)
            {
                throw new HttpResponseException(
                    System.Net.HttpStatusCode.NotFound);
            }
            return customer;
        }


        public async Task<IHttpActionResult> Post(Customer customer)
        {
            if (customer == null)
            {
                return BadRequest("No customer data provided");
            }
            var customerExists = await _Db.Customers.AnyAsync(c => c.Id == customer.Id);

            if (customerExists)
            {
                return BadRequest("Exists");
            }

            _Db.Customers.Add(customer);
            await _Db.SaveChangesAsync();
            return Ok();
        }


        public async Task<IHttpActionResult> Put(Customer customer)
        {
            if (customer == null)
            {
                return BadRequest("No customer data provided");
            }
            var existing = await _Db.Customers.FirstOrDefaultAsync(c => c.Id == customer.Id);

            if (existing == null)
            {
                return NotFound();
            }

            existing.Name = customer.Name;
            await _Db.SaveChangesAsync();
            return Ok();
        }


        public async Task<IHttpActionResult> Delete(int id)
        {
            var customer = await _Db.Customers.FirstOrDefaultAsync(c => c.Id == id);
            if (customer == null)
            {
                return NotFound();
            }
            _Db.Customers.Remove(customer);
            await _Db.SaveChangesAsync();
            return Ok();
        }
    }
}

```
## The database connection
I'm a big fan of Sql Server and entity framework on Mono can perfectly connect to Sql Server but... unfortunately not to an Azure Sql Server database. 
That's a pitty, so I needed to switch to MySql which works perfectly.
We need following entity framework related code:

```language-csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace OwinMonoDocker
{

    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext()
            : base("myMysqldb")
        {
        }
        public IDbSet<Customer> Customers { get; set; }
    }
    
    public class ApplicationDbInitializer : CreateDatabaseIfNotExists<ApplicationDbContext>
    {
        protected override void Seed(ApplicationDbContext context)
        {
            base.Seed(context);
            context.Customers.Add(new Customer { Name = "Microsoft" });
            context.Customers.Add(new Customer { Name = "Google" });
            context.Customers.Add(new Customer { Name = "Apple" });
        }
    }
}

```

For the above, following NuGet packages are used:
```language-markup
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="EntityFramework" version="6.1.2" targetFramework="net451" />
  <package id="Microsoft.AspNet.WebApi.Client" version="5.2.4-alpha1-150223" targetFramework="net451" />
  <package id="Microsoft.AspNet.WebApi.Core" version="5.2.4-alpha1-150223" targetFramework="net451" />
  <package id="Microsoft.AspNet.WebApi.Owin" version="5.2.4-alpha1-150223" targetFramework="net451" />
  <package id="Microsoft.AspNet.WebApi.OwinSelfHost" version="5.2.4-alpha1-150223" targetFramework="net451" />
  <package id="Microsoft.Owin" version="2.0.2" targetFramework="net451" />
  <package id="Microsoft.Owin.Host.HttpListener" version="2.0.2" targetFramework="net451" />
  <package id="Microsoft.Owin.Hosting" version="2.0.2" targetFramework="net451" />
  <package id="MySql.Data" version="6.8.4" targetFramework="net451" />
  <package id="MySQL.Data.Entities" version="6.8.3.0" targetFramework="net451" />
  <package id="Newtonsoft.Json" version="6.0.4" targetFramework="net451" />
  <package id="Owin" version="1.0" targetFramework="net451" />
</packages>
```
## The sources
The sources are available on [github](https://github.com/paulvanbladel/AspNet5-Web-Api-Sample/tree/blog-sample)
## What's next

This post was simply kind of preparation for the next big step: introducing DOCKER.
Stay tuned !

