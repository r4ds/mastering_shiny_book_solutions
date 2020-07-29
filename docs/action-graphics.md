# Graphics

<!--html_preserve--><div class="TODO">
This chapter is in development...
</div><!--/html_preserve-->

### Exercise 7.5.1 {-}

Make a plot with click handle that shows all the data returned in the input.

:::solution
#### Solution {-}

We can use the `allRows` argument in `nearPoints` to see all the data, and adds a boolean column that returns `TRUE` for the point that was clicked on


```r
library(shiny)
library(ggplot2)

ui <- fluidPage(
    plotOutput("plot", click = "plot_click"),
    tableOutput("data")
)
server <- function(input, output, session) {
    output$plot <- renderPlot({
        ggplot(mtcars, aes(wt, mpg)) + geom_point()
    }, res = 96)

    output$data <- renderTable({
        nearPoints(mtcars, input$plot_click, allRows = TRUE)
    })
}

shinyApp(ui = ui, server = server)
```
:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 7.5.2 {-}

Make a plot with click, dblclick, hover, and brush output handlers and nicely display the current selection in the sidebar. Plot the plot in the main panel.

:::solution
#### Solution {-}

We can use the `nearPoints` function to extract the data from `plot_click`, `plot_dbl`, and `plot_hover`. We need to use the function `brushedPoints` to extract the points within the `plot_brush` area.


```r
library(shiny)
library(ggplot2)

ui <- fluidPage(

    sidebarPanel(width = 6,
        h4("Selected:"),
        tableOutput("click"),
        h4("Double Clicked:"),
        br(),
        tableOutput("dbl"),
        h4("Hover:"),
        br(),
        tableOutput("hover"),
        br(),
        h4("Brushed:"),
        tableOutput("brush")
    ),

    mainPanel(width = 6,
      plotOutput("plot", 
                 click = "plot_click", 
                 dblclick = "plot_dbl",
                 hover = "plot_hover", 
                 brush = "plot_brush")
    )

)
server <- function(input, output, session) {
    
    output$plot <- renderPlot({
        ggplot(mtcars, aes(wt, mpg)) + geom_point()
    }, res = 96)

    output$click <- renderTable(nearPoints(mtcars, input$plot_click))
    output$hover <- renderTable(nearPoints(mtcars, input$plot_hover))
    output$dbl <- renderTable(nearPoints(mtcars, input$plot_dbl))
    output$brush <- renderTable(brushedPoints(mtcars, input$plot_brush))
}

shinyApp(ui = ui, server = server)
```
:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 7.5.3 {-}

Compute the limits of the distance scale using the size of the plot.

:::solution
#### Solution {-}


```r
output_size <- function(id) {
  reactive(c(
    session$clientData[[paste0("output_", id, "_width")]],
    session$clientData[[paste0("output_", id, "_height")]]
  ))
}
```
:::
