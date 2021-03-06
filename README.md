# economistdata
How to create charts in R The Economist-style


Re-create The Economist graph using ggplot2

So I was looking for some materials for ggplot2 and found a Workshop note from Harvard- Data Sciences Services. (http://tutorials.iq.harvard.edu/R/Rgraphics/Rgraphics.html)

They start with an amazing graph from the Economist and state that we will be able to recreate the graph at the end of the workshop using GGPLOT2. Even though the materials in the workshop are not enough (not even close) to create a chart like this, it is still an amazing materials for those who want to explore ggplot2 in R.

 
##Required Packages

library(ggrepel) #Add point labels
## Warning: package 'ggrepel' was built under R version 3.3.3
## Loading required package: ggplot2
## Warning: package 'ggplot2' was built under R version 3.3.3
library(ggplot2) #Main package for graph
library(ggthemes)#Themes for formating
## Warning: package 'ggthemes' was built under R version 3.3.3
library(extrafont)#Adding more font format (this package is optional because it will took sometimes for install all the font)
## Registering fonts with R
library(grid) #Add grid line
library(cowplot) #Add annotation
## Warning: package 'cowplot' was built under R version 3.3.3
## 
## Attaching package: 'cowplot'
## The following object is masked from 'package:ggplot2':
## 
##     ggsave
The Economist dataset
Let’s start with the dataset. It can be found in the package provided by the Harvard workshop. (http://tutorials.iq.harvard.edu/R/Rgraphics.zip)

library(readr)
## Warning: package 'readr' was built under R version 3.3.3
EconomistData <- read_csv("C:/Users/todu6001/Downloads/EconomistData.csv")
## Warning: Missing column names filled in: 'X1' [1]
## Parsed with column specification:
## cols(
##   X1 = col_integer(),
##   Country = col_character(),
##   HDI.Rank = col_integer(),
##   HDI = col_double(),
##   CPI = col_double(),
##   Region = col_character()
## )
View(EconomistData)
Let’s take a look at it.

EconomistData[1:6]
## # A tibble: 173 × 6
##       X1     Country HDI.Rank   HDI   CPI            Region
##    <int>       <chr>    <int> <dbl> <dbl>             <chr>
## 1      1 Afghanistan      172 0.398   1.5      Asia Pacific
## 2      2     Albania       70 0.739   3.1 East EU Cemt Asia
## 3      3     Algeria       96 0.698   2.9              MENA
## 4      4      Angola      148 0.486   2.0               SSA
## 5      5   Argentina       45 0.797   3.0          Americas
## 6      6     Armenia       86 0.716   2.6 East EU Cemt Asia
## 7      7   Australia        2 0.929   8.8      Asia Pacific
## 8      8     Austria       19 0.885   7.8      EU W. Europe
## 9      9  Azerbaijan       91 0.700   2.4 East EU Cemt Asia
## 10    10     Bahamas       53 0.771   7.3          Americas
## # ... with 163 more rows
Overall it’s a clean dataset, only some minor modification on the Region column and we’re good to go.

EconomistData$Region <- factor(EconomistData$Region,
                     levels = c("EU W. Europe",
                                "Americas",
                                "Asia Pacific",
                                "East EU Cemt Asia",
                                "MENA",
                                "SSA"),
                     labels = c("OECD",
                                "Americas",
                                "Asia &\nOceania",
                                "Central &\nEastern Europe",
                                "Middle East &\nnorth Africa",
                                "Sub-Saharan\nAfrica"))
Basic graph
Let’s start with the basic plot. We tell ggplot that EconomistData is our data and specify variables on each axis. We then add the data point with default color by region using geom_point.

graph1 <-  ggplot(EconomistData, aes (x=CPI, y=HDI))
graph1 + geom_point(aes(color = Region))


Well, not even close! Let’s start by listing out all the components we need to re-create the Economist’s graph based on what we have.
* Change the point shape and color
* Add a curve fit line
* Add gridlines
* Label some specific points
* Fix axis tick and labels.
* Fix axis scales
* Move legend to the top and change color
* Remove legend title
* Remove vertical grid line
* Add footnote

Point modification
We can change the shape of data point by using shape argument. The different points shapes commonly used in R are illustrated in the figure below :


We can see that shape 21 is an circle with border and color inside. I am gonna use that shape because the border’s thickness can be modified and we can fill the color with white to match with original graph. Let’s add shape=21 and fill with white color.

graph1 + geom_point(aes(color = Region),
                    shape=21, 
                    fill= "White")


It looks better however, it seems like the point border is smaller and thinner than the actual point.
We can use size to change point size and stroke to change border size.

g2 <- graph1 + geom_point(aes(color = Region),
                          shape=21, 
                          fill= "White",
                          size =3, 
                          stroke=1.5)
                        
ggdraw(g2)


Superb!!!

Fit line
By looking at the original graph, it seems like the line is created by a quadratic function y = log(x). The materials from Havard workshop use geom_line to add a linear regression fit line however, in this case, i will use geom_smooth since it’s not a linear relationship.

g2 + geom_smooth(method = "lm", 
                 formula = y ~log(x))


It seems to have a weird shaded area around the curve line. It represents the standard error for each predicted value. Let’s remove it by adding se=FALSE.

g2 + geom_smooth(method = "lm",
                 formula = y ~log(x), 
                 se=FALSE)


Finally, we proceed to change color and make sure the line is solid.

g3 <- g2 + geom_smooth(aes(fill="red"),method = "lm", 
                       formula = y ~log(x),
                       se=FALSE, 
                       linetype=1 , 
                       color= "Red") 

ggdraw(g3)


Labeling point
The ggplot has a built in function that allows us to label the data point.

g3 + geom_text(aes(label = Country))


Yep! Literally all data points are there and it’s a mess. There are only some specific data points that need to be labelled, unfortunatly, we have to manually identify and assign it.

point_1 <- c( "Venezuela", "Iraq", "Myanmar", "Sudan",
                    "Afghanistan", "Congo", "Greece", "Argentina", 
                    "India", "Italy",
                    "Botswana", "Cape Verde", "Bhutan", "Rwanda", "France",
                    "United States",  "Britain", "Barbados", "Norway", 
                    "New Zealand", "Singapore")
point_2 <- c("Russia","Brazil","Spain","Germany", "Japan","China","South Africa")
point_3 <-  c( "Venezuela", "Iraq", "Myanmar", "Sudan",
               "Afghanistan", "Congo", "Greece", "Argentina", 
               "India", "Italy",
               "Botswana", "Cape Verde", "Bhutan", "Rwanda", "France",
               "United States",  "Britain", "Barbados", "Norway", 
               "New Zealand", "Singapore","Russia","Brazil","Spain","Germany", "Japan","China","South Africa")
There is only some countries that have a connect line to the point. Therefore, I seperated them to see if I can do anything about it.

Now let’s label all the points without connection line.

g3 + geom_text(data=EconomistData[EconomistData$Country %in% point_1,], 
               aes(label=Country))


It looks alright but the labels overlapping the data point is annoying. I tried some formatting option in geom_text but I couldn’t solve the problem. And I found the package ggrepel which allow us to seperate label and data point as well as adding connection line.

g4 <- g3 + geom_text_repel(data=EconomistData[EconomistData$Country %in% point_1,],
                     aes(label=Country))
It looks way better. Now we proceed to label the data point with connection line.

g4 + geom_text_repel(data=EconomistData[EconomistData$Country %in% point_2,],
                     aes(label=Country))


It seems like 2 geom_text_repel codes create some messy overlap. I might put them together in one group or find a way to hack around it.
geom_text_repel has some options that allow us to adjust the distance between label and data point using box.padding. Let’s try that.

g5 <- g4 + geom_text_repel(data=EconomistData[EconomistData$Country %in% point_2,],
                     aes(label=Country),
                     box.padding = unit(1.75, 'lines'))
ggdraw(g5)


Still some small overlap but looks pretty neat. I am not completely satistfy with this labelling option so if any one have any better idea please let me know.

Legend box and color
It’s time to play around with the legend box and color.

We can change the color of the legend using scale_color_manual.The color code are based on the default HTML color code which can be found anywhere on google. Here is the link that I used.
(http://html-color-codes.info/)

 g6 <- g5 +  scale_color_manual( values = c("#23576E", "#099FDB", 
                                            "#29B00E", "#208F84", 
                                            "#F55840", "#924F3E")) +
      scale_fill_manual(name='My Lines', values=c("red"),labels=c("R^2=52%"))
It was a little bit tricky to get a seperate legend box for “R2=52%” value. I have to manually added it so that it will appear next to the region legend box. Next we proceed to move the Legend box to the top using theme and legend.position command.

g6 + theme(legend.position="top")


The legend box is in correct position but we want it to be in one line without any title. Note that guides function is to force the legend box to spread all the titles inside, so they stay in one row.

g7 <- g6 + theme(legend.position="top",
                 legend.title = element_blank(),
                 legend.box = "horizontal" ,
                 legend.text=element_text(size=8.5)) +
  guides(col = guide_legend(nrow = 1))
ggdraw(g7)


Grid line
In ggplot2 there are two types of gridlines: major and minor. Major gridlines emanate from the axis ticks while minor gridlines do not. Thus we need to hide the vertical gridlines, both major and minor, while keeping the horizontal major gridlines intact and change their color to grey. Since gridlines are theme items, to change their apperance you can use theme() and set the item with element_line() or if you want to remove the item completely, element_blank().

g8 <- g7 + theme(panel.grid.minor = element_blank(), 
        panel.grid.major = element_line(color = "gray50", size = 0.5),
        panel.grid.major.x = element_blank(),
        panel.background = element_blank(),
        line = element_blank())
ggdraw(g8)


X-axis and Y axis
The default X-axis spans from 0.2 to 1, incremental by 0.2 and Y-axis spans from 1 to 7.5, incremental by 2.5 . We will force the X-axis to span from 0.2 to 1 incremental by 0,1 and Y-axis to span from 1 to 10 incremental by 1 by setting the limits and breaks in scale_x_continous and scale_y_continous. We can also attach the title for both of them.

g9 <- g8 + scale_x_continuous(expand = c(0, 0),
                        limits=c(-.2,10.2),
                        breaks=seq(0,10,1), 
                        name = "Corruption Perception Index (10=Least corrupt)") +
  scale_y_continuous(expand = c(0, 0),
                     limits=c(0.2,1),
                     breaks=seq(0.2,1,0.1), 
                     name = "Human Development Index,2011 (1=best)")
ggdraw(g9)


We also want to remove the axis tick and make some font format for the axis’s titles.

g10 <-g9 +theme(axis.ticks.length = unit(.15, "cm"),
        axis.ticks.y = element_blank(),
        axis.title.x = element_text(color="black", 
                                    size=10,
                                    face="italic"),
        axis.title.y = element_text(color="black",
                                    size=10,
                                    face="italic"))
ggdraw(g10)


Title and Footnote
Adding title is simple. The ggtitle will do the work just fine and we use theme to add some format.

g11 <- g10+ ggtitle("Corruption and human development\n") + 
  theme(plot.title = element_text(hjust = -0.15, 
                                  vjust=2.12, 
                                  colour="black",
                                  size = 14,
                                  face="bold"))
ggdraw(g11)


The footnote is a little more tricky to create. Some people tried to extract a png file of the graph and use grid package to manually draw it into the png. However, it’s too complicated and took sometimes to make perfect adjustment.

After some intense google searching, I found a package called cowplot and it has a feature add_sub that allows us to add a footnote directly into graph.

g12 <-  add_sub(g11,"Source:Transparency International; UN Human Development report",
                x = -0.07,
                hjust = 0,
                fontface = "plain",
                size= 10) 
ggdraw(g12)


VOILA!!!! Now let’s take a look at the original graph. They look really close beside the fact that I wasn’t able to find a way to move the legend position and my axis lines look uglier. However, I was surprised because I can get this close to the original graph using R only.


Wrap-up
This looks at first a simple chart to make, but it turns out to be more complex than I expected.It required more than just standard knowledge in gglot2.

Finally, the point isn’t that you can mimic other styles. It’s that there’s enough flexibility in R to create your own chart, eventhough some of them seems impossible to create. With some customization, we can ultilize R to fullfill our creativity and create amazing looking chart.

Source
[1] http://tutorials.iq.harvard.edu/R/Rgraphics/Rgraphics.html
