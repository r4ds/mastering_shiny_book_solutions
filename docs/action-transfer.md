# Uploads and Downloads

<!--html_preserve--><div class="TODO">
This chapter is in development...
</div><!--/html_preserve-->

### Exercise 9.4.1 {-}

Use the ambient package by Thomas Lin Pedersen to generate [worley noise](https://ambient.data-imaginist.com/reference/noise_worley.html) and download a PNG of it.

:::solution
#### Solution {-}

A general method for saving a `PNG` is to select the `png` driver using the function `png()`. The only argument the driver needs is the name of the plot (this will be stored relative to your current working directory!). You will not see the plot when running the `plot` function because it is being saved to the file instead. When we're done plotting, we used the `dev.off()` command to close to the connection to the driver.


```r
library(ambient)
noise <- ambient::noise_worley(c(100, 100))

png("noise_plot.png")
plot(as.raster(normalise(noise)))
dev.off()
```
:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 9.4.2 {-}

Create an app that lets you upload a csv file, select a variable, and then perform a `t.test()` on that variable. After the user has uploaded the csv file, you'll need to use `updateSelectInput()` to fill in the available variables. See Section 10.1 for details.

:::solution
#### Solution {-}

We can use the `fileInput` widget and the `accept` argument to ensure our app only takes on `.csv`s. In the `server` function we'll select the `datapath` from within our file input and use this to update our `selectInput`. We need to put the `updateSelectInput` within a reactive because we need the options to change if the user selects another file. Lastly, we use `input$variable` to create our t-test output.


```r
library(shiny)

ui <- fluidPage(
    sidebarLayout(
        sidebarPanel(
            fileInput("file", "Select CSV", accept = ".csv"), # file widget
            selectInput("variable", "Select Variable", choices = NULL) # select widget
        ),
        mainPanel(
           verbatimTextOutput("results") # t-test results
        )
    )
)

server <- function(input, output,session) {

    # get data from file
    data <- reactive({
        req(input$file)
        read.csv(input$file$datapath)
    })

    # create the select input based on the numeric columns in the dataframe
    observeEvent( input$file, {
        num_cols <- dplyr::select_if(data(), is.numeric)
        updateSelectInput(session, "variable", choices = colnames(num_cols))
    })

    # print t-test results
    output$results <- renderPrint({
        req(!is.null(input$variable))
        t.test(data()[input$variable])
        })
}

shinyApp(ui = ui, server = server)
```
:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 9.4.3 {-}

Create an app that lets the user upload a csv file, select one variable, draw a histogram, and then download the histogram. For an additional challenge, allow the user to select from .png, .pdf, and .svg output formats.

:::solution
#### Solution {-}

Adapting the code from the example above, rather than print a t-test output, we create a plot reactive to use in the UI's `output$results` and to use within our `downloadHander`. We can use the `ggsave` function to switch between `input$extension` types.


```r
library(shiny)
library(ggplot2)

ui <- fluidPage(
    tagList(
        br(), br(),
        column(4,
               wellPanel(
                   fileInput("file", "Select CSV", accept = ".csv"), # file widget
                   selectInput("variable", "Select Variable", choices = NULL), # select widget
               ),
               wellPanel(
                   radioButtons("extension", "Save As:", choices = c("png", "pdf", "svg"), inline = TRUE),
                   downloadButton("download", "Save Plot")
               )
             ),
        column(8, plotOutput("results"))
    )
)

server <- function(input, output,session) {

    # get data from file
    data <- reactive({
        req(input$file)
        read.csv(input$file$datapath)
    })

    # create the select input based on the numeric columns in the dataframe
    observeEvent( input$file, {
        num_cols <- dplyr::select_if(data(), is.numeric)
        updateSelectInput(session, "variable", choices = colnames(num_cols))
    })

    # plot histogram
    plot_output <- reactive({
        req(!is.null(input$variable))

        ggplot(data()) +
            aes_string(x = input$variable) +
            geom_histogram()
        })

    output$results <- renderPlot(plot_output())

    # save histogram using downloadHandler and plot output type
    output$download <- downloadHandler(
        filename = paste0("histogram.", input$extension),
        content = function(file){
            ggsave(file, plot_output(), device = input$extension)
        }
    )

}

shinyApp(ui = ui, server = server)
```
:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 9.4.4 {-}

Write an app that allows the user to create a Lego mosaic from any .png file using Ryan Timpe's brickr package. Once you've completed the basics, add controls to allow the user to select the size of the mosaic (in bricks), and choose whether to use “universal” or “generic” colour palettes.

:::solution
#### Solution {-}

Instead of limiting our file selection to a `csv` as above, here we are going to limit our input to a `png`. We'll use the `png::readPNG` function to read in our file, and specify the size and color of our mosaic in `brickr`'s `image_to_mosaic` function. Read more about the package and examples [here](https://github.com/ryantimpe/brickr)


```r
library(shiny)
library(brickr)
library(png)

ui <- fluidPage(
    sidebarLayout(
        sidebarPanel(
            fluidRow(
                fileInput("myFile", "Choose a file", accept = c('image/png')),
                sliderInput("size", "Select Size:", min = 1, max = 100, value = 35),
                radioButtons("color", "Select Color Palette:", choices = c("universal", "generic"))
            )
        ),
        mainPanel(plotOutput("result"))
    )
)


server <- function(input, output) {

    observeEvent(input$myFile, {
        inFile <- input$myFile
        if (is.null(inFile))
            return()

        output$result <- renderPlot({
            png::readPNG(inFile$datapath) %>%
                image_to_mosaic(img_size = input$size, color_palette = input$color) %>%
                build_mosaic()
        })
    })

}
shinyApp(ui = ui, server = server)
```
:::