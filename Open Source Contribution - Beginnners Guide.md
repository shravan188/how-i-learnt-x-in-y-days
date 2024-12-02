# Guide to Contribute to Open Source

A detailed step by step guide for absolute beginners to contribute to open source

## Step 0 : Develop a learners mindset
Learning anything takes time, so develop a hunger to learn new things. Also patience and perseverance are key for long term success


## Step 1 : Learn a programming language
Learn any language of your choice, like Python, Java, Javascript, Golang, etc. Practice problems until you are familiar with basic syntax and concepts like data types, conditional statements, loops etc. There are multiple resources to practice on Github, like 30 Days of x, etc

## Step 2: Choose an open source repo which is looking for contributers and in the lanaguage above
If you are a beginner, it is preferable to choose smaller repos with stars in the 10 to 150 range. Also choose a repo with good community i.e. has an active Discord/Slack channel where you can go to to ask for help if stuck. Some places where you can get such repos are
1. GSoC and similar open source events
2. Reddit forums like r/OpenSource

## Step 3: Scan the codebase - a top down approach
Once you select a repo, go through all the folders (and some files within each folder). List down the **key libraries** used in the project (should take around 1 to 2 hours). For example a simple Python codebase might have the following key libraries
* Fastapi
* Sqlalchemy
* Pydantic


## Step 4: Learn each library - a bottom up approach
Take any one of the libraries listed in Step 3. Do a quickstart tutorial from the docs or a short video (less than 30 mins, not more), to understand the basic concepts. Then scan the codebase for every line belonging to that library, and if you encounter any function/line which you do not understand, search for it in the docs (or google/youtube). Then try to implement that function as an addition to the quickstart code. Do this until you cover all the lines related to that library in the codebase
Repeat this for all the other libraries as well. Try to cover all libraries relevant to the issue you are trying to solve.

## Step 5: Choose an issue and create a good PR
By the end of step 4, you would have a good idea of the codebase. Use that knowledge to select a relevant issue and create a good pr following all the coding standards.


## Things to Remember
* Make detailed notes of each and every thing you learn. Prefarably keep a notepad/text file open before you start learning and write each and every point you learn into that notepad, including doubts you faced and the resources you referred to
* Do not ask simple doubts to open source maintainers. Ask a doubt only if you are badly stuck and not able to find a solution inspite of trying every potential solution
