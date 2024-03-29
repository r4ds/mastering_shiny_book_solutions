# Bookmarking


```{=html}
<div class="TODO">
This chapter is in development...
</div>
```

### Exercise 11.3.1 {-}

Generate app for visualising the results of [noise::ambient_simplex()](https://ambient.data-imaginist.com/reference/noise_simplex.html). Your app should allow the user to control the frequency, fractal, lacunarity, and gain, and be bookmarkable. How can you ensure the image looks exactly the same when reloaded from the bookmark? (Think about what the seed argument implies).

:::solution
#### Solution {-}

For this example, we'll use the bookmarking by setting `enableBookmarking = "url"` within the `shinyApp` function. In order to ensure the simulation is the same each time we re-render the bookmark we'll create a grid to use for points then set the seed to `42` to ensure the same image is rendered when the bookmark is loaded.


```r
library(shiny)
library(ambient)

ui <- function(request) {
    fluidPage(
    sidebarLayout(
        sidebarPanel(
            sliderInput("frequency", "Frequency", min = 0, max = 1, value = 0.01),
            selectInput("fractal", "Fractal", choices = c("none", "fbm", "billow", "rigid-multi")),
            sliderInput("gain", "Gain", min = 0, max = 1, value = 0.5),
        ),
        mainPanel(
            plotOutput("result")
        )
    )
  )
}


server <- function(input, output, session) {

    grid <- long_grid(seq(1, 10, length.out = 1000), seq(1, 10, length.out = 1000))

    noise <- reactive({
        ambient::gen_simplex(
            x = grid$x,
            y = grid$y,
            seed = 42,
            frequency = input$frequency,
            fractal = input$fractal,
            gain = input$gain
        )
    })

    output$result <- renderPlot({
        plot(grid, noise())
    })

    observe({
        reactiveValuesToList(input)
        session$doBookmark()
    })
    onBookmarked(updateQueryString)
}

shinyApp(ui = ui, server = server, enableBookmarking = "url")
```
:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 11.3.2 {-}

Make a simple app that lets you upload a csv file and then bookmark it. Upload a few files and then look in shiny_bookmarks. How do the files correspond to the bookmarks? (Hint: you can use readRDS() to look inside the cache files that Shiny is generating).

:::solution
#### Solution {-}

By setting the `state$values$data` equal to the `data` reactive, we can store the contents of the uploaded `csv`. Looking in the `shiny_bookmarks` folder we see an `input.rds` which has the same 4 arguments as `input$file`:

 1. `name`
 2. `size`
 3. `type`
 4. `datapath`

All of these except the `datapath` are the same as when we upload the file; rather than the temporary location the file is saved to within the shiny session, the datapath becomes `0.csv`, a csv file created within the same folder are our `input.RDS`.


```r
library(shiny)

ui <- function(request){
    fluidPage(
        sidebarLayout(
            sidebarPanel(
                bookmarkButton(),
                fileInput("file", "Choose CSV File", multiple = TRUE,accept = ".csv")
                ),
            mainPanel(
                tableOutput("contents")
            )
        )
    )
}


server <- function(input, output) {

    # create reactive of input file
    data <- reactive({
        req(input$file)
        read.csv(input$file$datapath)
    })

    # display head
    output$contents <- renderTable( head(data()) )

    # set the state to the df reactive
    onBookmark(function(state){
        state$values$data <- data()
    })

    # on restore set df to the state
    onRestore(function(state){
        data <- reactive(state$values$data)
    })
    enableBookmarking(store="server")
}

shinyApp(ui, server)
```
:::
