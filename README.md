# A little background  
I am a craft beer enthusiast and love to sample new beers (especially West Coast IPAs, Lagers and Saisons).  A while ago some friends and I found a guy that distributes craft beer mainly from California.  How we order is quite simple, we send a request for information and this guy sends us a PDF file with what’s in stock.  I was appointed as the official transcriber.  It is my duty to transcribe the catalog to an Excel file.  Everyone fills out what beer they want on the Excel file and then we place the order, we pay, and after a couple of days there’s always something new in my fridge.
At first, I gladly fulfilled my appointed duty, but after a couple of months I found it extremely tedious to perform this monotonous and repetitive task.  That’s when I remembered that a while ago, I helped a retail customer to automate how he ingested the general nutrition facts labels into his product catalog using Azure Form Recognizer (AFR for short).  Since our beer provider always uses the same PDF file format it made complete sense to use AFR again, only that this time I would need to get my hands dirtier writing some code in python.
Here is a look at how a product description looks like in the product catalog shared by our provider.

![Sample Catalog](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/sample%20catalog.png)

I fired up my AFR resource and my first big surprise is that they have a few more prebuilt models into it.  They used to have receipts and business cards, but there are now prebuilt models for invoices, contracts, vaccination cards, and more.  Obviously, the ability to build your own model is still there.
My second, and greatest surprise, is that we now have an AFR Studio!  Previously we relied on a sample opensource app built on TypeScript that some hero developer(s) created.  You could use the sample app online (not knowing if some change on it will break your project) or download the code and execute it locally on your infrastructure.
![Azure Form Recognizer Main](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/AFR%20main.png)
 
# How to train your model
It is quite easy.  You need a couple of sample documents, they can be a variety of file types, either PDFs or images. Select Custom Model, create a new project.  Give it a name, link it to an AFR resource and feed him the sample files.
![Azure Form Recognizer New Model](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/AFR%20new%20project.png)
 
And from there it is just some light mouse and keyboard work, select text and type the kind of label you want to assign to that area on the document.  This is where the data labeling happens.  AFR is not only an OCR tool, but it also helps you build key values.  For instance, if you do an OCR on the image below you wouldn’t know if “King Al” is the name of the brewery, the beer, the style, or if it’s part of the label on the image of the can.  That’s where AFR shines.  If you have a standard document “form” where you know the position of every label (key) you can identify the text related to it (value).
![Training Model](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/AFR%20train.png)
 
On the same image I used my mouse to select the text “Equilibrium collab Toppling Goliath Brewing” and assign it the label “brewery”.  I did that for the rest of the general information located on the right side of the PDF file.  You can see the different colors highlighted on the image matching a label on the far-right side for “brewery”, “beer”, “style”, “abv”, “untappd” and “price”.  If you take a closer look at the image, I intentionally ignored the words “King Al” and “Equilibrium” that appear on the image of the can at the center of the screen.  AFR requires as few as 5 different files to train a model, but I used around 25 different ones for my model.  As this is assisted training the more documents you use and variations of them the more your model will learn.
You might guess that depending on the complexity of your document and the number of documents used to train is how much time you will spend on this activity.  In my case, considering just six labels I spent around 15 to 20 minutes to label the documents.
After that I clicked on “Train” on the upper right of the screen, the model started training and after a short while – maybe 5 or 10 minutes – I got my new model.
![Azure Form Recognizer Models](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/AFR%20models.png) 

From there you can test your model
![Azure Form Recognizer Test Model](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/AFR%20test.png) 

You can get a good idea of how effective it is.  Part of the functionality of the service is to provide you with a certainty value.  On the image above you can see on the far-right side what the model discovered and how sure it is that he is correct.  Right now, my model is fluctuating between the low 70s to high 80s percent.  It is not a perfect model, it can be improved with more samples and further training, but I’ll take it for now.  Later I can retrain it and replace the existing one without breaking my code.
Speaking of which.
#Code
There’s a limitation with AFR, it can analyze multipage forms, but it can’t analyze the same form collated multiple times inside the same document.  And that’s exactly how I get the product catalog.  So, to automate the whole data input process (the annoying transcription of the PDF file to an Excel file) I decided to do some stuff with Python. 
I’m more of a T-SQL, MDX, DAX guy, so I was looking forward to working with Python and some APIs and launched my VSCode to get my hands dirty.  The first time I worked with AFR a couple of years ago I used Postman to make sure that I got the APIs calls right.  I did that again this time since I am using a newer version of the service.  Besides, Postman has a very handy feature that auto creates code snippets in different languages.  This greatly reduced the amount of time I spent on VSCode. 

![Postman Screenshot](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/postman.png) 

I’ll give you the GitHub repository, but first let me take you for a spin on what I did.
![Visual Studio Code 1](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/code%201.png) 

I used PyPDF to help me split the original catalog into individual documents to be able to analyze each one.  After that I defined two different functions – Inference and GetInference – the way AFR works is asynchronous, so you request a document to be analyzed and then you come back for the results.
So, the first function, “Inference” receives as a parameter the filepath (in this case a PDF file) and uses the API endpoint, key and defines the headers.  We fire up the API call and we get back a request id.  We will use this request id in the next function “GetInference” to come back and retrieve the result of the analysis.  
I introduced an artificial delay on this API call since, as mentioned above, the whole analyzing and data gathering is asynchronous.  After getting the response I drop it in a Pandas DataFrame and scrub it up a bit.
Finally, call the functions looping through all the different files created during the PDF split and export the data to an Excel file.
![Visual Studio Code 2](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/code%202.png) 

In conclusion, after fooling around a bit with Python to split the pdf file and testing the AFR the results are exactly what is expected, no longer transcribing the whole product catalog. Neat.  :D 
![Excel Results Sample](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/excel%20results.png)

![Catalog Sample vs Excel 1](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/results%20samples%201.png) ![Catalog Sample vs Excel 2](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/results%20samples%202.png) ![Catalog Sample vs Excel 3](https://github.com/ElPilot13/AFR-BeerSample/blob/main/img/results%20samples%203.png) 
   

# Final comments  
AFR provides you with an SDK but I decided to go the API route in order to teach myself different ways to work with the service and just because I wanted to :D
The SDK takes care of the asynchronous requests, error handling, retrying, etc, so if you go that way your development time should decrease.
This is not meant to be a definite guide or a step by step on how to use AFR.  You can use AFR in a variety of different ways and complement it or use it to complement extra services.  I could have easily done all this with Azure Logic Apps or Azure Functions, but I wanted to write some code for this pet project.  AFR can be used to enrich a Cognitive Search Pipeline as a custom skill.  The output could have been a relational database or a NoSQL repository just like my previous customer.  I once helped another customer use AFR to automate the ingestion of handwritten forms (yes, AFR is based on Computer Vision so he can understand handwritten text – I can’t promise if he will be able to transcribe your doctor’s prescription) so that they could be fed to a machine learning model for fraud detection.  Let your imagination run wild. 

