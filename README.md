# cloudscribe.Web.Navigation
ASP.NET Core MVC solution for navigation menus and breadcrumbs.

[![Join the chat at https://gitter.im/joeaudette/cloudscribe](https://badges.gitter.im/joeaudette/cloudscribe.svg)](https://gitter.im/joeaudette/cloudscribe?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

This was implemented in support of a larger project [cloudscribe.Core.Web](https://github.com/joeaudette/cloudscribe/) but has been moved to a separate repository since it has no dependencies on other "cloudscribe" components and should be useful in any ASP.NET Core MVC project that needs menus and breadcrumbs. I've also used it to implement navigation in my [cloudscribe SimpleContent](https://github.com/joeaudette/cloudscribe.SimpleContent) project, as well as for the navigation in my cloudscribe.Core project.


What is currently provided is a NavigationViewComponent which is used with various views to produce menus and breadcrumbs. The NavigationViewComponent depends on INavigationTreeBuilder to provide the tree of navigation nodes. We have an XmlNavigationTreeBuilder and a JsonNavigationTreeBuilder which can build the navigation tree from the corresponding file type. I find it easier to work with an xml file in this case because it is easier to find the beginning and end tag and comments are supported, one misplaced comma can easily break the json file and comments are not supported there.

I think when you have controllers and actions corresponding to the things you want navigation for, using an xml file is a nice easy way to wire up the menu. Menu nodes can be filtered from display by roles, so administrative or member only navigation items can be integrated into a holistic navigation tree.

It is currently possible to have nested TreeBuilders, for example in the [cloudscribe SimpleContent](https://github.com/joeaudette/cloudscribe.SimpleContent) project, we used an XmlTreeBuilder for the root navigation but it invokes another [custom treebuilder](https://github.com/joeaudette/cloudscribe.SimpleContent/blob/master/src/cloudscribe.SimpleContent.Services/PagesNavigationTreeBuilder.cs) that adds the navigation nodes for SimpleContent pages. You can see how I plugged it into the [navigation.xml](https://github.com/joeaudette/cloudscribe.SimpleContent/blob/master/src/example.WebApp/navigation.xml) file in that project.

There is also support for memory caching, but support for distributed cache has not been implemented yet.

You can download/clone this repo and run the NavigationDemo.Web project to see a working example.

## Installation

Prerequisites:

*  [Visual Studio 2015](https://www.visualstudio.com/en-us/downloads) 
*  [ASP.NET 5 RC1 Tooling](https://get.asp.net/) 

To install from nuget.org open the project.json file of your web application and in the dependencies section add:

    "cloudscribe.Web.Navigation": "1.0.0-*"
    
Visual Studio 2015 should restore the package automatically, you could also open a command prompt and use dnu restore in your project folder.

Unfortunately it is not yet possible for us to install the needed views from nuget. So for the moment you need to manually [copy the views folder and views from here](https://github.com/joeaudette/cloudscribe.Web.Navigation/tree/master/src/cloudscribe.Web.Navigation/content)

In your Startup.cs you will need this at the top: 

    using Microsoft.Framework.DependencyInjection.Extensions;
    using cloudscribe.Web.Navigation;

and in ConfigureServices you will need this:

    services.TryAddScoped<INavigationTreeBuilder, XmlNavigationTreeBuilder>();
    services.TryAddScoped<INodeUrlPrefixProvider, DefaultNodeUrlPrefixProvider>();
    services.TryAddScoped<INavigationNodePermissionResolver, NavigationNodePermissionResolver>();
    services.Configure<NavigationOptions>(Configuration.GetSection("NavigationOptions"));
    services.Configure<DistributedCacheNavigationTreeBuilderOptions>(Configuration.GetSection("DistributedCacheNavigationTreeBuilderOptions"));
    services.Configure<MemoryCacheNavigationTreeBuilderOptions>(Configuration.GetSection("MemoryCacheNavigationTreeBuilderOptions"));
    services.TryAddScoped<INavigationCacheKeyResolver, DefaultNavigationCacheKeyResolver>();

actually those last 3 items related to caching are not fully implemented yet

In your _ViewImports.cshtml file add:

    @using cloudscribe.Web.Navigation

You also need an navigation.xml file to define the navigation nodes. This can go in the root folder of your web app (not in wwwroot)

    <?xml version="1.0" encoding="utf-16"?>
    <NavNode key="Home" parentKey="RootNode" controller="Home" action="Index" text="Home" isRootNode="true">
      <Children>
        <NavNode key="About" parentKey="RootNode" controller="Home" action="About" text="About">
          <Children />
        </NavNode>
        <NavNode key="Contact" parentKey="RootNode" controller="Home" action="Contact" text="Contact">
          <Children />
        </NavNode>
        <NavNode key="Members" parentKey="RootNode" controller="Home" action="Members" text="Members" viewRoles="Admins,Members">
          <Children />
        </NavNode>
        <NavNode key="Administration" parentKey="RootNode" controller="Home" action="Administration" text="Administration" viewRoles="Admins">
          <Children />
        </NavNode>
      </Children>
    </NavNode>
    
Now you can use the ViewComponent in your views.

For example if you started with the standard ASP.NET 5 project template, you will have hard coded html navigation in the _Layout.cshtml file like this:

    <div class="navbar-collapse collapse">
        <ul class="nav navbar-nav">
            <li><a asp-controller="Home" asp-action="Index">Home</a></li>
            <li><a asp-controller="Home" asp-action="About">About</a></li>
            <li><a asp-controller="Home" asp-action="Contact">Contact</a></li>
        </ul>
        @await Html.PartialAsync("_LoginPartial")
    </div>
  
  You would replace that with this:
  
      <div class="navbar-collapse collapse">
          @await Component.InvokeAsync("Navigation", "BootstrapTopNav", NamedNavigationFilters.TopNav) 
          @await Html.PartialAsync("_LoginPartial")
      </div>
  
  That makes the top bootstrap navigation, now to add breadcrumbs put this in at the indicated spot:
  
      <div class="container body-content">
            @await Component.InvokeAsync("Navigation", "BootstrapBreadcrumbs", NamedNavigationFilters.Breadcrumbs)
            @RenderBody()

The div and the @RenderBody() should already be there, you just add the middle part that invokes the breadcrumbs.

The NavigationDemo.Web project has examples such as nodes filtered by roles, and a way to adjust breadcrumbs from a controller action. For now if you want to see a more advanced/detailed use of cloudscribe.Web.Navigation, you can study how it is being used in [cloudscribe.Core](https://github.com/joeaudette/cloudscribe).

Follow me on twitter @cloudscribeweb and @joeaudette
