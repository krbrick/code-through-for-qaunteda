---
title: "Wordclouds, Lexical Dispersion and STM"
author: "Kara Brick"
date: "10/2/2020"
output:
  tufte::tufte_html: default
---
```{r}
library(tufte)
```
  
I borrowed heavily from the Caleb's blog: <em>https://blog.paperspace.com/intro-to-datascience/</em> to set up a wordcloud, the most visually appealing and straightforward of the qualitative data visualization, in my opinion.
For this example, I will be using data from a set that I have been working on and am familiar with: interviews from after a workshop that took place with a group teachers in Liberia. Liberians speak Liberian English, and that is not the same as what we speak, but I think most of the stopwords are the same. 

Start out by installing quanteda and RColorBrewer. I'm adding pander and dplyr because they will be useful, also. I'm setting my path here, too, in the knitr chunk.
```{r setup}
knitr::opts_chunk$set(root.dir = normalizePath("~/Desktop/CPP527"))

library(RColorBrewer)
library(pander)
library(dplyr)
```
Next I will load the data from excel by Importing in in R. I will now rename my dataset as raw, because the name is lengthy.   
The data can be separated by interviewee. Some of the language is different, and the cells represent answers that have already been split up into conveniant chunks for longhand qualitative coding.    
```{r, include = FALSE}
setwd("/Users/karabrick/Desktop/CPP527")
```

```{marginfigure}
"My role as a teacher is to make sure I teach our incoming generation well" - DC
```
This data is from excel, so it is a dataframe. I saved is as a csv file and had to make sure that all of the text from the interviews were in one cell in excel, and that those cells had a label that I could call for the text_field when creating a corpus in quanteda.  
We will start the use of quanteda by putting the data into the form of a corpus.   
Quanteda needs to read the text and transform it into a corpus. So, we need packages quanteda and readtext. Understanding how to accurately format these commands and the paths to get to them was what took me the most time on this.    
The text I am using does not necessarily have docvars - or document variables. The only variable assigned here is the Interviewee. I chose to use raw data for this one because getting into the structure of quanteda copora with excel files with qualitative codes of my own making seems too confusing for this code-through, and it helps me understand the functions better. When you look at the summary of my corpus, it is easy to see what I am talking about - there are ten documents (the interviews). There aren't dates, interviewers, hashtags or topics, like you might find with other useful texts.    

A good resource for understanding quanteda from the bottom up is from the University of Virginia Library: <em>https://data.library.virginia.edu/a-beginners-guide-to-text-analysis-with-quanteda/</em>   
Below I am importing my data, assigning a text_field and naming my documents after the interviewee initials.   

```{r}
library(quanteda)
library(readtext)
dat_inT <- read.csv(paste0("/Users/karabrick/Desktop/CPP527/IN.DAT.csv"))
int.corpus <- corpus(dat_inT, text_field = "Interview")
docnames(int.corpus) <- int.corpus$"Interviewee"
summary(int.corpus)
```



I had to take out a few more stopwords - 'yeah' and 'okay' which makes me laugh. I guess Liberian English utilizes these words more, because the first time I did it they were front and center in my word cloud.   



This is what my code ended up looking like to get a wordcloud. By the way, I had to do some funny stuff with the r calls at the top of r chunks to make it a full image in Tufte's layout. You can also make images appear in the margin.    
```{r,fig.fullwidth = TRUE}
#WordCloud
IC.col <- brewer.pal(11, "Spectral")
IC.cloud <- int.corpus %>% 
  dfm(remove = c('yeah', 'okay',stopwords('english')), remove_punct = TRUE) %>%
    dfm_trim(verbose = FALSE)

set.seed(100)
textplot_wordcloud(max_size = 4, IC.cloud, color = IC.col)
```


You can adjust word clouds by size, number of words, nice color scales in color brewer, etc. Below I set the rotation to zero, so that all the words would be horizontal.    




```{r, fig.margin = TRUE}
IC.col <- brewer.pal(9, "Greens")
IC.cloud <- int.corpus %>% 
  dfm(remove = c('yeah', 'okay',stopwords('english')), remove_punct = TRUE) %>%
    dfm_trim(verbose = FALSE)

set.seed(100)
textplot_wordcloud(max_size = 3, max_words = 50, rotation = 0, IC.cloud, color = IC.col)
```
   
A note on Color Brewer for R Markdown: You see where I circled the letters? Where the arrow is pointing? That is your R language. When you are selecting a color scheme, those are the letters you need for your brewer.pal function.    


   
```{marginfigure, echo = TRUE}
Each color palette has a maximum and a minimum number of colors. This is the first argument we call in our function brewer.pal. If you put the wrong number, your wordcloud will still generate with an errror directing you to change the number to the correct maximum number. 
```
![ColorBrewerScreenshot](/Users/karabrick/Desktop/CPP527/colorb.png)



You can check out Tufte magic here: <em>http://rstudio.github.io/tufte/envisioned/</em>. You can also open a Tufte template in a new R markdown file in R.     




    
Now for Lexical Dispersion. This is really just about visualizing where and how many times a word appears in a text. A different way of describing Lexical Dispersion would be to say the spread of words.   
The kwic function is case-insensitive, but to show the function char_tolower, I'm changing converting it to lower case here.  
This is a really simplified code using Quanteda's kwic (key words in context) function. Since this dataset is so simple, we can easily see that the word *challenges* is used in different proportions by different Interviewees. At the bottom of the plot, you see the axis is titled Relative Token Index. This is calculated by the word index number, or how many words along it occurs. A token is also known as a word. I assigned a scale of 'absolute' to the x-ray plot.    
By referencing my word cloud, I chose the word *challenges* and the phrase *mental health* because those are two important concepts in these interviews. I also wanted to demonstrate that phrases can be used, as well as multiple patterns.   
Let's check out a code.   


```{r, fig.margin = TRUE}
int.corpus_low <- char_tolower(int.corpus)

kwic(int.corpus_low, pattern = c("challenges", phrase("mental health"))) %>%
    textplot_xray(scale = "absolute")
```

Another cool function of kwic is words in context. For instance, I can choose a word, such as student, and ask kwic to bring up phrases around that word, with the 'window' argument being the number of words before or after that word.    
```{r, fig.margin = TRUE, eval = FALSE}
head(kwic(int.corpus_low, "student", window = 4))
```


In this section I will put my corpus into a document feature matrix (DFM), a useful class for quanteda.    
From here we can look at a list of tokens, or words, by frequency.    



```{r}
int.corpus_dfm <- dfm(int.corpus, remove_punct = TRUE, remove = c('yeah', 'okay', 'first', stopwords('english'), stem = TRUE))
head(int.corpus_dfm, nf = 50)
```


But wait! Apparently DFMs should be tokenized before becoming DFMs.   


```{r}
this.tokens.in <- tokens(int.corpus,
                     remove_numbers = TRUE,
                     remove_separators = TRUE,
                     remove_punct = TRUE)
this.tokens.in<- tokens_select(this.tokens.in, stopwords('english'), selection='remove')
this.tok.in.dfm <- this.tokens.in %>% dfm(remove = 'okay')
```




To take it one step further, exploration of topic modelling is in order.    
Topic modelling tries to unearth topics within documents and show much the document overall is related to those topics. It presupposes that words occuring more are of more relevance to a topic. It tries to show meaning through how many times which words are used, and groups both words and documents by their topics, or groups of words used often.    
To make use of this, we have to trim our dfm. Trimming takes out words that very rarely occur or occur all the time. At first, I did not trust this 'trimming'. I don't know why. So I ran stm without trimming, and all that came back were 'okay' 'yeah' and 'lesson'.     
```{r}
library("stm")
topic.count <- 6
tok.dfm.trim <- dfm_trim(this.tok.in.dfm, min_docfreq = 0.075, max_docfreq = 0.90, docfreq_type = "prop") # min 7.5% / max 95%
dfm2stm <- convert(tok.dfm.trim, to = "stm")
model.stm <- stm(dfm2stm$documents, dfm2stm$vocab, K = topic.count, data = dfm2stm$meta, init.type = "Spectral")
```
To keep from boring us all to tears, I'll keep my topic numbers and table outputs low. Useful visualizations including plotting the topics (Top Topics, to the right), and showing how they relate to their docs and each other and comparisons between topics, as in the 'perspectives' plot. 

```{r, fig.margin = TRUE}
data.frame(t(labelTopics(model.stm, n = 1)$prob))
plot(model.stm, type = "summary", text.cex = 0.5)
```
```{r, fig.fullwidth = TRUE}
plot(model.stm, n = 10, type = "perspectives", topics = c(1,5))

```
In this visualization, called the perspectives plot, word size is related to their expected use in the topics, both 1 and 5, which I chose beceause they are towards the top of the Top Topics plot. The vertical position of words is random. Along the x axis, the words are arrange by how much they are associated with each of the topics. So the word 'whenever' is more associated with topic 1 than the word 'appear', which is more towards the midde.    

These types of visualizations and ananlyses are really pivotal for open-ended, qualtitative data.   

Thanks for reading this breakdown of some cool quanteda functions.    