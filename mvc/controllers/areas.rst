Areas
=====
By `Tom Archer`

MVC uses a folder hierarchy pattern that defines the relationship between your controllers, views, and actions. You can see this with the default wizard-generated ASP.NET MVC app where you start with a ``Controllers`` folder containing several controllers (``Account``, ``Home``, and ``Manage``). If you open the ``HomeController.cs`` file, You'll note that the Home controller has three actions - ``Index``, ``About``, and ``Contact`` - that return a view object without qualifying which view to return. (A fourth action - ``Error`` - is present, but it explicitly names the location of the view to return.) So, how does MVC know which view to return when its location is not explicitly stated?

MVC finds the associated view for a controller action by searching in two folders: ``Views/[controller][action].cshtml`` and ``Views/Shared/[action].cshtml``. Therefore, when the ``HomeController.About`` method attempts to create a view without specifying the location of the view, the MVC routing engine will look for ``Views/Home/About.cshtml`` first. If that's not found, the routing engine will look for ``Views/Shared/About.cshtml``. If neither are found, an exception is thrown.

This pattern works well for many Web apps. However, for larger apps, this folder hierarchy can become difficult to manage as you have controllers in one parent folder, and views in a separate parent folder. This is where `Areas` come in. Areas enable you to partition your project such that controllers and their views can be stored under a parent folder that defines their logical grouping.

Let's take a look at an example to illustrate how Areas are created and used. In this example, we'll take a simple store app (MyStore) that has a several logical groupings of controllers and actions. These groupings are ``Services``, ``Products``, and ``Deals``. The following steps will walk you though creating an area for each grouping, as well as the controllers and views for each.

#. In Solution Explorer, right-click the project name and create a new folder named ``Areas``.
#. Right-click the ``Areas`` folder and create a new folder named ``Products``. This folder will house the controllers and views folders for the ``Products`` area.
#. Right-click the ``Areas/Products`` folder and create a new folder named ``Controllers``. This folder will house all the controllers for the ``Products`` area.
#. Right-click the ``Areas/Products/Controllers`` folder and add a new MVC controller named HomeController. By default, this controller will be created with a single action - ``Index``.
#. Right-click the ``Areas/Products`` folder and create a new folder named ``Views``. This folder will house all the views for the ``Products`` area.
#. Right-click the ``Areas/Products/Views`` folder and create a new folder named ``Home`` to match the controller name.
#. Right-click the ``Areas/Products/Views/Home`` folder and add a new MVC view with the same name as the ``Home`` action: ``Index``.

At this point, you should have a project folder hierarchy that includes the following:

- Project name

  - Areas

    - Products

      - Controllers

        - HomeController

      - Views

        - Home

          - Index.cshtml

To add the Services and Deals areas, you would simply repeat steps 2-7, replacing "Products" with the name of the area you're defining.

Once you've defined the folder hierarchy, you need to tell MVC that each controller is associated with an area. You do that by decorating the controller name with the ``[Area]`` attribute. Here's what the boiler-plate ``Products/HomeController`` looks like with the attribute added.

.. code-block:: c#
  :emphasize-lines: 4

  ...
  namespace MyStore.Areas.Products.Controllers
  {
      [Area("Products")]
      public class HomeController : Controller
      {
          // GET: /<controller>/
          public IActionResult Index()
          {
              return View();
          }
      }
  }

The final step is to set up a route definition that works with your newly created areas. The :doc:`routing` article goes into detail about how to create route definitions, including using conventional routes versus attribute routes. In this example, we'll use a conventional route. To do so, simply open the ``startup.cs`` file and modify it by adding the highlighted route definition below.

.. code-block:: c#
  :emphasize-lines: 4-6

  ...
  app.UseMvc(routes =>
  {
      routes.MapRoute(name: "areaRoute",
              template: "{area:exists}/{controller}/{action}",
              defaults: new { controller = "Home", action = "Index" });

      routes.MapRoute(
          name: "default",
          template: "{controller=Home}/{action=Index}");
  });

Now, when the user enters http://<yourApp>/products, the Index action method of the ``HomeController`` in the ``Products`` area will be invoked and its view will be sent to the client for rendering.

Linking between areas
---------------------

Once you've set up your areas, you'll need to know how to link between areas. By way of example, take a look at the following hierarchy.

- Project name

  - Areas

    - Products

      - Controllers

        - HomeController

      - Views

        - Home

          - Index.cshtml

  - Controllers

    - HomeController

  - Views

    - Home

      - Index

As you can see, this example has two ``HomeController`` controllers; one that's defined in an area (``Products/Controllers/HomeController``) and one that's defined outside an area (``Controllers/HomeController``). Let's say you want to put a link on each page that links back to the other page.

Linking from outside to inside
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Linking from outside to inside simply means to link  from a place that's outside of any area to an action that's inside an area. For example, linking from ``Controllers/HomeController`` to ``Products/Controllers/HomeController``. To do this, you could simply use the ``Html.ActionLink`` method and specify the area as an anonymous object.

.. code-block:: c#

  @Html.ActionLink("See Products Home Page", "Index", "Home", new { area = "Products" }, null)

Linking from inside to outside
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Linking from inside to outside simply means to link from a place that's inside an area to an action that's outside an area. As you can probably guess, you simply specify an area of "".

.. code-block:: c#

  @Html.ActionLink("Go to Home Page", "Index", "Home", new { area = "" }, null)

Summary
-------
Areas are a very useful tool for grouping semantically-related controllers and actions under a common parent folder. In this article, you learned how to set up your folder hierarchy to support ``Areas``, how to specify the ``[Area]`` attribute to denote an controller as belonging to a specified area, and how to define your routes with areas. Finally, you saw how to link between areas - including linking from outside to inside, and linking from inside to outside.
