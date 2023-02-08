# Functions


```{=html}
<div class="TODO">
This chapter is in development...
</div>
```

### Exercise 14.4.1 {-}

The following app plots user selected variables from the msleep dataset for three different types of mammals (carnivores, omnivores, and herbivores), with one tab for each type of mammal. Remove the redundancy in the `selectInput()` definitions with the use of functions.


```r
library(tidyverse)

ui <- fluidPage(
  selectInput(inputId = "x",
              label = "X-axis:",
              choices = c("sleep_total", "sleep_rem", "sleep_cycle", 
                          "awake", "brainwt", "bodywt"),
              selected = "sleep_rem"),
  selectInput(inputId = "y",
              label = "Y-axis:",
              choices = c("sleep_total", "sleep_rem", "sleep_cycle", 
                          "awake", "brainwt", "bodywt"),
              selected = "sleep_total"),
  tabsetPanel(id = "vore",
              tabPanel("Carnivore",
                       plotOutput("plot_carni")),
              tabPanel("Omnivore",
                       plotOutput("plot_omni")),
              tabPanel("Herbivore",
                       plotOutput("plot_herbi")))
)

server <- function(input, output, session) {

  # make subsets
  carni <- reactive( filter(msleep, vore == "carni") )
  omni  <- reactive( filter(msleep, vore == "omni")  )
  herbi <- reactive( filter(msleep, vore == "herbi") )

  # make plots
  output$plot_carni <- renderPlot({
    ggplot(data = carni(), aes_string(x = input$x, y = input$y)) +
      geom_point()
  }, res = 96)
  output$plot_omni <- renderPlot({
    ggplot(data = omni(), aes_string(x = input$x, y = input$y)) +
      geom_point()
  }, res = 96)
  output$plot_herbi <- renderPlot({
    ggplot(data = herbi(), aes_string(x = input$x, y = input$y)) +
      geom_point()
  }, res = 96)

}

shinyApp(ui = ui, server = server)
```

:::solution
#### Solution {-}

We can see a pattern here where we are creating the same type of plot for each tabset panel, with the only variable changing being the `vore` argument. We can reduce everything we see in triplicate to functions! We can use `map` to create a single `create_panels` function which will create a tab for each of our `species`. On the server side, the data is filtered three times, and the plots are created three times. We can create a single rendering function that given the correct string it will filter the data, create the correct plot, and assign it to the correct output.


```r
library(tidyverse)

# use a vector for function inputs
species <- c("Carnivore", "Omnivore", "Herbivore")

# educe to a single UI function
# Marly: this didn't work!
create_panels <- function(id) {
    tabPanel(id, plotOutput(paste0("plot_", id)))
}

ui <- fluidPage(
    selectInput(inputId = "x",
                label = "X-axis:",
                choices = c("sleep_total", "sleep_rem", "sleep_cycle",
                            "awake", "brainwt", "bodywt"),
                selected = "sleep_rem"),
    selectInput(inputId = "y",
                label = "Y-axis:",
                choices = c("sleep_total", "sleep_rem", "sleep_cycle",
                            "awake", "brainwt", "bodywt"),
                selected = "sleep_total"),
    tabsetPanel(
        tabPanel("Carnivore", plotOutput("plot_Carnivore")),
        tabPanel("Omnivore", plotOutput("plot_Omnivore")),
        tabPanel("Herbivore", plotOutput("plot_Herbivore"))
    )
    # this works without the tabsetPanel function - why?!
    # purrr::map(species, create_panels)
)

server <- function(input, output, session) {

    # rendering plot function for each panel
    render_outputs <- function(id) {
        output[[paste0("plot_", id)]] <- renderPlot({
            msleep %>%
                filter(vore == tolower(stringr::str_remove(id, "vore")) %>%
                ggplot() +
                aes_string(x = input$x, y = input$y) +
                geom_point()
            )
        })
    }

    # apply to the species vector using map
    purrr::map(species, render_outputs)
}

shinyApp(ui = ui, server = server)
```
:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 14.4.2 {-}

Continue working with the same app from the previous exercise, and further remove redundancy in the code by modularizing how subsets and plots are created.

:::solution
#### Solution {-}

TODO: I'm unsure what to do with this one since we haven't yet introduced modules? 

:::

<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->
<!---------------------------------------------------------------------------->

### Exercise 14.4.3 {-}

Suppose you have an app that is slow to launch when a user visits it. Can
modularizing your app code help solve this problem? Explain your reasoning.

:::solution
#### Solution {-}

No, we're just packaging our code into neater functions - this doesn't change or optimize what is loaded when the app is launched. In fact, modularizing might even make your application slower in some cases. 
:::
