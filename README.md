// Model/Data.cs
using System;

namespace CloudBasedApp.Models
{
    public class DataModel
    {
      public Guid Id { get; set;}
        public string Message { get; set; }
        public DateTime CreatedAt {get; set;}
    }
}

// Controllers/DataController.cs
using Microsoft.AspNetCore.Mvc;
using CloudBasedApp.Models;
using System;
using System.Collections.Generic;
using System.Linq;

namespace CloudBasedApp.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class DataController : ControllerBase
    {
        private static List<DataModel> _dataStore = new List<DataModel>(); // In-memory storage for demo purposes

        // POST: api/Data
        [HttpPost]
        public IActionResult Post([FromBody] DataModel data)
        {
           if(data == null || string.IsNullOrEmpty(data.Message)){
              return BadRequest("Invalid data provided");
           }

            data.Id = Guid.NewGuid();
            data.CreatedAt = DateTime.UtcNow;
            _dataStore.Add(data);
            return CreatedAtAction(nameof(Get), new { id = data.Id }, data);
        }

        // GET: api/Data
        [HttpGet]
        public IActionResult Get()
        {
          return Ok(_dataStore);
        }


       // GET: api/Data/id
        [HttpGet("{id}")]
        public IActionResult Get(Guid id) {
            var item = _dataStore.FirstOrDefault(x => x.Id == id);
            if (item == null)
             {
                 return NotFound($"Item with id: {id} not found");
              }
             return Ok(item);
       }

        // DELETE: api/Data/id
        [HttpDelete("{id}")]
        public IActionResult Delete(Guid id)
        {
             var itemToDelete = _dataStore.FirstOrDefault(x => x.Id == id);
             if(itemToDelete == null)
             {
                   return NotFound($"Item with id: {id} not found");
             }

            _dataStore.Remove(itemToDelete);
             return NoContent();
       }
    }
}

// Program.cs
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;

namespace CloudBasedApp
{
  public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
    }
}

// Startup.cs
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace CloudBasedApp
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

         // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
             services.AddControllers();
             services.AddEndpointsApiExplorer();
            services.AddSwaggerGen();
        }


        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
                 app.UseSwagger();
                app.UseSwaggerUI();
            }

             app.UseRouting();
            app.UseAuthorization();


            app.UseEndpoints(endpoints =>
            {
              endpoints.MapControllers();
            });

        }
    }
}
