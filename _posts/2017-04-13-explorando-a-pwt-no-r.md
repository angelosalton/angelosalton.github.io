---
title: Explorando a Penn World Table com R e shiny
tags:
  - R
---

A Penn World Table é uma base de dados que contém variáveis relevantes de desenvolvimento econômico para países ao redor do mundo. Esse post apresenta uma construção em *shiny*, um pacote que possibilita desenvolver aplicações web a partir do R. O objetivo foi montar uma ferramenta para comparar países visualmente, com dados anuais de 1980 a 2014.

O código-fonte:

```r
library(dplyr) # para manipular os dados
library(shiny)
library(plotly) # pacote com opções de gráficos
library(pwt9) # carrega a Penn World Table 9

pwt <- pwt9.0 %>%
  filter(year >= 1980)

# dando nomes aos bois
colnames(pwt)[5:47] <- c("Expenditure-side real GDP at chained PPPs (in mil. 2011US$)", 
                         "Output-side real GDP at chained PPPs (in mil. 2011US$)", "Population (in millions)", 
                         "Number of persons engaged (in millions)", "Average annual hours worked by persons engaged", 
                         "Human capital index, based on years of schooling and returns to education; see Human capital in PWT9.", 
                         "Real consumption of households and government, at current PPPs (in mil. 2011US$)", 
                         "Real domestic absorption, (real consumption plus investment), at current PPPs (in mil. 2011US$)", 
                         "Expenditure-side real GDP at current PPPs (in mil. 2011US$)", 
                         "Output-side real GDP at current PPPs (in mil. 2011US$)", "Capital stock at current PPPs (in mil. 2011US$)", 
                         "TFP level at current PPPs (USA=1)", "Welfare-relevant TFP levels at current PPPs (USA=1)", 
                         "Real GDP at constant 2011 national prices (in mil. 2011US$)", 
                         "Real consumption at constant 2011 national prices (in mil. 2011US$)", 
                         "Real domestic absorption at constant 2011 national prices (in mil. 2011US$)", 
                         "Capital stock at constant 2011 national prices (in mil. 2011US$)", 
                         "TFP at constant national prices (2011=1)", "Welfare-relevant TFP at constant national prices (2011=1)", 
                         "Share of labour compensation in GDP at current national prices", 
                         "Average depreciation rate of the capital stock", "Exchange rate, national currency/USD (market+estimated)", 
                         "Price level of CCON (PPP/XR), price level of USA GDPo in 2011=1", 
                         "Price level of CDA (PPP/XR), price level of USA GDPo in 2011=1", 
                         "Price level of CGDPo (PPP/XR),  price level of USA GDPo in 2011=1", 
                         "0/1/2: relative price data for consumption, investment and government is extrapolated (0), benchmark (1) or interpolated (2)", 
                         "0/1/2: relative price data for exports and imports is extrapolated (0), benchmark (1) or interpolated (2)", 
                         "0/1: the exchange rate is market-based (0) or estimated (1)", 
                         "0/1: the observation on pl_gdpe or pl_gdpo is not an outlier (0) or an outlier (1)", 
                         "Correlation between expenditure shares of the country and the US (benchmark observations only)", 
                         "Statistical capacity indicator (source: World Bank, developing countries only)", 
                         "Share of household consumption at current PPPs", "Share of gross capital formation at current PPPs", 
                         "Share of government consumption at current PPPs", "Share of merchandise exports at current PPPs", 
                         "Share of merchandise imports at current PPPs", "Share of residual trade and GDP statistical discrepancy at current PPPs", 
                         "Price level of household consumption,  price level of USA GDPo in 2011=1", 
                         "Price level of capital formation,  price level of USA GDPo in 2011=1", 
                         "Price level of government consumption,  price level of USA GDPo in 2011=1", 
                         "Price level of exports, price level of USA GDPo in 2011=1", 
                         "Price level of imports, price level of USA GDPo in 2011=1", 
                         "Price level of the capital stock, price level of USA in 2011=1"
)

# INTERFACE -----------------------
ui <- fluidPage(
   
   # Título
   titlePanel("Penn World Table 9"),
   
   # Barra lateral
   sidebarLayout(
      sidebarPanel(width = 3,
        selectInput(inputId = "vars", label = "Variable", choices = colnames(pwt)[5:47]),
        selectInput(inputId = "year", label = "Year", choices = unique(pwt$year), selected = 2014)
      ),
      # Show a plot
      mainPanel(
         plotlyOutput("plot", height = "600px")
      )
   )
)

# SERVER ---------------------
server <- function(input, output){
  
  # database update
  pw <- reactive({
    a <- pwt %>% filter(year == input$year)
    return(a)
  })
  
  # light grey boundaries
  l <- list(color = toRGB("grey"), width = 0.5)
  
  # specify pwt projection/options
  g <- list(
    showframe = FALSE,
    showcoastlines = FALSE,
    projection = list(type = 'Mercator')
  )
  
  output$plot <- renderPlotly({
  plot_geo(pw()) %>%
    add_trace(
      z = ~get(input$vars), values = ~get(input$vars), colors = 'Blues',
      text = ~country, locations = ~isocode, marker = list(line = l)
    ) %>%
      layout(geo = g)
  })
}
# Run the application 
shinyApp(ui = ui, server = server)
```

[Acesse aqui para ver o resultado.](https://salton.shinyapps.io/pwtmap/)
