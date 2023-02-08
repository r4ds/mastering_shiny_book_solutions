# Modules


```{=html}
<div class="TODO">
This chapter is in development...
</div>
```

### Exercise 15.6.1 {-}

Example passing input$foo to reactive and it not working.

:::solution
#### Solution {-}

I don't really know what this question is asking, but I think the point is to remember:

> The main challenge with this sort of code is remembering when you use the reactive (e.g. x\$value) vs. when you use its value (e.g. x\$value()). Just remember that when passing an argument to a module, you want the module to react to the value changing which means that you have to pass the reactive, not it's current value.

Where in this scenario, `input$foo` is analogous to `x$value`.
:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 15.6.2 {-}

Rewrite `selectVarServer()` so that both data and filter are reactive. Pair it with a app function that lets the user pick the dataset with the dataset module, a function with an `inputSelect()` that lets the user filter for numeric, character, or factor variables.

:::solution
#### Solution {-}

The modules `datasetInput`, `datasetServer`, and `selectVarInput` are the same, as well as the `find_vars` function. 

We can start by creating `selectFilterInput` which has the filtering options as choices, and `selectFilterServer` which returns the filtering function given the selected choice string. 


```r
# create a filter selection input
selectFilterInput <- function(id) {
    selectInput(NS(id, "filter"), "Filter",
                choices = c("Numeric", "Character", "Factor"),
                selected = "Numeric")
}

# switch the function to be applied within the server
selectFilterServer <- function(id) {
    moduleServer(id, function(input, output, session) {
        eventReactive(input$filter, {
            switch(input$filter,
               "Numeric" = is.numeric,
               "Character" = is.character,
               "Factor" = is.factor
               )
        })
    })
}
```

Now we can update the `selectVarServer` to take on an additional `filter` argument, and change the update function to not only observe when the `data` reactive changes but also our new `filter` widget changes. Lastly we pass in the filter reactive to the `find_vars` function.



```r
selectVarServer <- function(id, data, filter) { # filter argument
    moduleServer(id, function(input, output, session) {
        observeEvent({
            data()
            filter() #observe changes in filter reactive
            }, {
            updateSelectInput(session, "var", choices = find_vars(data(), filter())) # filter as reactive
        })
        reactive(data()[[input$var]])
    })
}
```

Putting it together, we add our new module to the UI and server, and by saving the result of the `selectFilterServer` to `filt` we can pass that to the `selectVarServer`


```r
selectVarApp <- function() {
    ui <- fluidPage(
        datasetInput("data", is.data.frame),
        # call the new filter UI
        selectFilterInput("filter"),
        selectVarInput("var"),
        verbatimTextOutput("out")
    )
    server <- function(input, output, session) {
        data <- datasetServer("data")
        # store the filtering function as a reactive
        filt <- selectFilterServer("filter")
        # pass the reactive to the select module
        var <- selectVarServer("var", data, filter = filt)
        output$out <- renderPrint(var())
    }

    shinyApp(ui, server)
}
```
:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 15.6.3 {-}

The following code defines output and server components of a module that takes a numeric input and produces a bulleted list of three summary statistics. Create an app function that allows you to experiment with it. The app function should take a data frame as input, and use `numericVarSelectInput()` to pick the variable to summarise.


```r
summaryOuput <- function(id) {
  tags$ul(
    tags$li("Min: ", textOutput(NS(id, "min"), inline = TRUE)),
    tags$li("Max: ", textOutput(NS(id, "max"), inline = TRUE)),
    tags$li("Missing: ", textOutput(NS(id, "n_na"), inline = TRUE))
  )
}

summaryServer <- function(id, var) {
  moduleServer(id, function(input, output, session) {
    rng <- reactive({
      req(var())
      range(var(), na.rm = TRUE)
    })

    output$min <- renderText(rng()[[1]])
    output$max <- renderText(rng()[[2]])
    output$n_na <- renderText(sum(is.na(var())))
  })
}
```

:::solution
#### Solution {-}

We only need to add the code above to the `selectVarApp()` example in the book, and adapt the app code to include the `summaryOutput` instead of the `verbatimTextOutput`, and on the server side pass `var` to the `summaryServer` function instead of to the text output. 


```r
selectVarApp <- function(filter = is.numeric) {
    ui <- fluidPage(
        datasetInput("data", is.data.frame),
        selectVarInput("var"),
        summaryOutput("summary")
    )
    server <- function(input, output, session) {
        data <- datasetServer("data")
        var <- selectVarServer("var", data, filter = filter)
        summaryServer("summary", var)
    }

    shinyApp(ui, server)
}

selectVarApp()
```
:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 15.6.4 {-}

The following module input provides a text control that lets you type a date in ISO8601 format (yyyy-mm-dd). Complete the module by providing a server function that uses output$error to display a message if the entered value is not a valid date. The module should return a Date object for valid dates. (Hint: use strptime(x, "%Y-%m-%d") to parse the string; it will return NA if the value isn't a valid date.)


```r
ymdDateUI <- function(id, label) {
  label <- paste0(label, " (yyyy-mm-dd)")

  fluidRow(
    textInput(NS(id, "date"), label),
    textOutput(NS(id, "error"))
  )
}
```

:::solution
#### Solution {-}

We create a `ymdDateServer` function that renders the error if `strptime(input$date, "%Y-%m-%d")` is `NA`. 


```r
ymdDateServer <- function(id, label) {
    moduleServer(id, function(input, output, session) {
        output$error <- renderText({
            print(input$date)
            print(strptime(input$date, "%Y-%m-%d"))
            if (!is.na(strptime(input$date, "%Y-%m-%d")) | input$date == "") {
                NULL
            } else {
               "Entered value is not a proper date"
            }

        })
    })
}
```

We put the `UI` and `Server` code together in the `ymdApp` function below:


```r
ymdApp <- function(filter = is.numeric) {
    ui <- fluidPage(
        ymdDateUI("ymd", "Time")
    )
    server <- function(input, output, session) {
        ymdDateServer("ymd")
    }
    shinyApp(ui, server)
}
ymdApp()
```
:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 15.6.5 {-}

In `radioExtraServer()`, return a list that contains both the value and whether or not it came from other.

:::solution
#### Solution {-}

We can adapt the reactive we return from `radioExtraServer` to return both the reactive and whether it came from the primary button choices or not as a list. 


```r
radioExtraServer <- function(id) {
    moduleServer(id, function(input, output, session) {
        observeEvent(input$other, ignoreInit = TRUE, {
            updateRadioButtons(session, "primary", selected = "other")
        })
        
        selected <- reactive({
            if (input$primary == "other") {
                input$other
            } else {
                input$primary
            }
        })

        # return the selected reactive inside a list
        # adding whether it came from primary or not 
        list(selected =
                 reactive({
                     if (input$primary == "other") {
                         input$other
                     } else {
                        input$primary
                     }
                 }),
             primary =
                 reactive(input$primary != "other")
             )
    })
}
```

In doing so, we need to adapt the `radioExtraApp` code to return `extra$selected()` rather than `extra`.


```r
radioExtraApp <- function(...) {
    ui <- fluidPage(
        radioExtraUI("extra", NULL, ...),
        textOutput("value")
    )
    server <- function(input, output, server) {
        extra <- radioExtraServer("extra")
        output$value <- renderText({
            paste0("Selected: ", extra$selected())
        })
    }

    shinyApp(ui, server)
}
radioExtraApp(c("a", "b", "c"))
```
:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 15.6.6 {-}

In `wizardServer()` verify that the namespacing has been set up correctly by using two or more wizards in a single add, and checking that you can navigate through each wizard independently.

:::solution
#### Solution {-}


:::
