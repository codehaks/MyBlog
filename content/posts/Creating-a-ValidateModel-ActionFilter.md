---
title: "How to work with Action Filters in ASP Core"
date: 2018-04-18T11:10:14+04:30
draft: false
---

Every time you need to store some data in database you have to validate it first. we usually need to create an if/else block and make sure ModelState is valid and send back some error messages if it's not.

ActionFilters are a great way to implement the DRY principle. Here we are going to create a simple ActionFilter to do the Model Validation before getting into the Action.

The code goes like this :

```Csharp
public class ValidateModelAttribute : Attribute, IActionFilter
    {
    public void OnActionExecuted(ActionExecutedContext context)
        {
            //throw new NotImplementedException();
        }
        public void OnActionExecuting(ActionExecutingContext context)
        {
            if (!context.ModelState.IsValid)
            {
                var ctrl = (Controller)context.Controller;
                var view = new ViewResult
                {
                    ViewName = context.RouteData.Values["Action"].ToString(),
                    ViewData = ctrl.ViewData
                };
                context.Result = view;
            }
        }
    }
```

We want to Validate model before running the Action so we have to implement **OnActionExecuting** method from IActionFilter interface. First we use **ActionExecutingContext** to find out if ModelState is valid or not.

If model state is valid then we pass the data to action and everything goes as planned there, But if it's not valid we have to redirect user back to original View containing the same ViewData.

To do that I used **context.Controller** and cast it back to **Controller** then used **Context** to extract action name from route data. This way our filter works with any action in any controller. otherwise we hd to hard core the view name.

At the end by setting up **context.Result** we are telling ASP Core MVC pipeline to skip the rest of the code and generate the view from here.

### Conclusion

ActionFiter is a great way to remove repetitive code and make it more elegant. it's easier to maintain and less work to re-use. You should remember that ActionFilters run after ModelBinding procedures so if you need to stop pipeline from ModelBinding you need to use other filters.
