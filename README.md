## Editable DataTables for shiny apps.

**Author:** Jason Bryer, Ph.D.
**Email:** jason@bryer.org

Use the `devtools` package to install the development version of `DTedit`:

```r
devtools::install_github('jbryer/DTedit')
```

The `dtedit_demo` will run a sample `shiny` app with to editable data tables.

```r
DTedit::dtedit_demo()
```

![DTedit Screen Shot](inst/screens/dtedit_books_edit.png)

#### Getting Started with `DTedit`

You can download a simple shiny app using `DTedit` here: [inst/template/app.R](inst/template/app.R)

There are three steps to using `DTedit` in your shiny application.

1. Define callback function for inserting, updating, and deleting data.

```r
my.insert.callback <- function(data, row) {
	mydata <- rbind(data, mydata)
	return(mydata)
}

my.update.callback <- function(data, olddata, row) {
	mydata[row,] <- data[1,]
	return(mydata)
}

my.delete.callback <- function(data, row) {
	mydata[row,] <- NULL
	return(mydata)
}
```

Typically these functions would interact with a database. As written here, the data would be lost between shiny sessions.

2. Create the `dtedit_server` object within your `server` function. 

```r
DTedit::dtedit_server(
	   id = 'mycontacts',
	   thedata = mydata,
	   edit.cols = c('name', 'email', 'useR', 'notes'),
	   edit.label.cols = c('Name', 'Email Address', 'Are they an R user?', 'Additional notes'),
	   input.types = c(notes='textAreaInput'),
	   view.cols = c('name', 'email', 'useR'),
	   callback.update = my.update.callback,
	   callback.insert = my.insert.callback,
	   callback.delete = my.delete.callback)
```

The `id` parameter defines the name of the object shiny tables withavailable to the `uiOutput`. The `thedata` is a `data.frame` for the initial view of the data table. This can be an empty (i.e. no rows) `data.frame`. The structure of the `data.frame` will define the inputs (e.g. `factor`s will be drop down, `Date` will use `dateInput`, `numeric`s will use `numericInput`, etc.). There are many other parameters to custom the behavior of `dtedit`, see `?dtedit` for the full list.

3. Use `dtedit_ui` in your UI to display the editable data table.

The `id` you will pass to `dteditui` is the name you passed to the `dtedit_server` created on the server side.

```r
dtedit_ui('mycontacts')
```


## Include plots to demonstrate editable shiny tables

Wouldn't it be more exciting if we can add plots that change according to editable tables in a shiny app! To do this, you only need to prepare functions for your plots and/or shiny widget arguements that control the plots. Here I will show an example to have interactive plots with editable table using the data above, and you can find the code of the shiny app at https://github.com/lynceuslq/DTedit/blob/master/R/DTedit_demo_with_plots.R

1. Create plot functions 
```r
publ <- function(dataframe, list) {
  with(list, {
  #### here is the content of your plot code, please just edit this part of the function
    df <- subset(dataframe, Publisher==type) #example plotting code
    ggplot() + geom_bar(data=df, mapping = aes(y=Date)) #example plotting code
  ####
    
  })
}
```
In the plot function above, `dataframe` coorespounds to the content in the editable table, `list` represents all arguements in the `input` from your shiny UI, which can be passed to the your plot functions. You can replace the code marked up in the function above to make any plot you want, but in order to make things work, the function should follow the exact format. And please note that you should just call your inputs from UI directly instead of using `input$` or `input[[]]` in your plot function. 

2. Create a list to add widgets to control plots in the shiny app 

Here are the widgets I want to include in th UI
```r
selectInput("type", 
            "type to plot", 
            c("Wiley", "Wadswort Internation Group", "Wadsworth & Brooks", 
                                                    "Springer"), "Springer")
						    )				   

```

To include this in the shiny app, the code need to be warpped up in a list below, in which the sequence of the elements in the list shoud follow their exact order in the correspounding shiny widget function. And please note that you need to avoid using "add", "edit", "remove" and "copy" as your widget ID, as they are already engaged by DTedit widgets.

```r
mywidgets <- list(list("selectInput", ### define the type of your widget
                       "type", ### input id
                       "type to plot", ### label 
                       c("Wiley", "Wadswort Internation Group", "Wadsworth & Brooks", 
                                                    "Springer"), "Springer")
                  )
```

3. Call `dtedit()` with plot functions and new widgets 
This is the simplest step, just add the vectors you created above to your `dtedit()` function in server as options.
```r
dtedit(input, output,
         name = 'books',
         thedata = books,
         pltfunc = publ,
         input.args = mywidgets,
         edit.cols = c('Title', 'Authors', 'Date', 'Publisher'),
         edit.label.cols = c('Book Title', 'Authors', 'Publication Date', 'Publisher'),
         input.types = c(Title='textAreaInput'),
         input.choices = list(Authors = unique(unlist(books$Authors))),
         view.cols = names(books)[c(5,1,3)],
         callback.update = books.update.callback,
         callback.insert = books.insert.callback,
         callback.delete = books.delete.callback)
	 
```
After calling your shiny app, you can see your plot showing below the table and it will change accordingly to the table.
