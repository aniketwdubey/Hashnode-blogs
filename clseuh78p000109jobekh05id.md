---
title: "Building a Simple Blogging GraphQL API with Django and Strawberry"
seoTitle: "GraphQL API with Django and Strawberry"
seoDescription: "GraphQL API for managing a book database using Django and Strawberry GraphQL"
datePublished: Fri Feb 09 2024 16:11:08 GMT+0000 (Coordinated Universal Time)
cuid: clseuh78p000109jobekh05id
slug: building-a-simple-blogging-graphql-api-with-django-and-strawberry
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1707494712071/67b9d25f-eb9e-4a90-9edc-12ee21828a0a.png
tags: django, apis, graphql, rest-api, strawberry

---

## **What is GraphQL?**

GraphQL is a query language for APIs that enables clients to request only the data they need and nothing more. Unlike traditional REST APIs, where clients are often constrained by the fixed structure of the responses, GraphQL allows clients to specify the shape of the response they require, empowering them with more flexibility and efficiency.

With GraphQL, clients can send queries to the server specifying exactly which fields they want to retrieve, eliminating over-fetching and under-fetching of data. This declarative approach to data fetching results in more efficient network usage and faster response times, making GraphQL an ideal choice for building modern APIs.

## **Introduction**

In this article, we'll explore how to build a Django-based blogging GraphQL API that focuses on providing basic functionalities for a blogging platform. It allows users to create posts and add comments, all accessible through a GraphQL endpoint. This project aims to demonstrate how to leverage Django and Strawberry to rapidly develop a GraphQL API for a blogging platform.

MyBlogProject leverages the power of GraphQL to provide a seamless and intuitive interface for interacting with the blogging platform. With GraphQL, clients can craft queries tailored to their specific needs, enabling a more efficient exchange of data between the client and server.

## **1\. Project Structure**

Code : [https://github.com/aniketwdubey/myblogproject](https://github.com/aniketwdubey/myblogproject)

The project structure follows a typical Django setup:

`blog/`: Contains the Django app for blogging functionalities.

`myblogproject/`: Main project directory.

`db.sqlite3`: SQLite database file.

`manage.py` : Django's command-line utility for administrative tasks.

## **2\. Features**

* **Post Creation**: Users can create new blog posts with titles, content, and author information.
    
* **Commenting**: Users can add comments to existing blog posts.
    
* **GraphQL API**: The API is implemented using the Strawberry framework, providing a GraphQL endpoint for interacting with the blog data.
    

Users can update and delete the posts and comments as well!

## **3\. Installation**

1. Clone the repository:
    
    ```bash
    git clone https://github.com/yourusername/myblogproject.git
    ```
    
2. Navigate to the project directory:
    
    ```bash
    cd myblogproject
    ```
    
3. Create a virtual environment:
    
    ```bash
    python -m venv venv
    ```
    
4. Activate the virtual environment:
    
    * On Windows:
        
        ```bash
        venv\Scripts\activate
        ```
        
    * On macOS and Linux:
        
        ```bash
        source venv/bin/activate
        ```
        
5. Install dependencies:
    
    ```bash
    pip install -r requirements.txt
    ```
    
6. Apply migrations:
    
    ```bash
    python manage.py makemigrations
    python manage.py migrate
    ```
    

## **2\. Usage**

1. Run the development server:
    
    ```bash
    python manage.py runserver
    ```
    
2. Access the GraphQL endpoint at [`http://localhost:8000/graphql/`](http://localhost:8000/graphql/) to interact with the API.
    

## 3\. CRUD Operations using GraphQL API

▪︎ query posts

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707493753405/69fb7727-7611-4e50-a91b-95b5dabc0ceb.png align="center")

▪︎ create post

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707493798062/e5458217-c32e-4dfe-8ac5-dbb3e5bdb796.png align="center")

▪︎ create another post

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707493821682/955a8f84-cf94-4460-9aab-76fbfef3e329.png align="center")

▪︎ query all posts

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707493851739/00781234-becb-4d8c-8bc8-4904e66962eb.png align="center")

▪︎ update post with postId:13

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707493872146/050ed037-d5f9-45c0-8f88-2c222d432a36.png align="center")

▪︎ query all posts again

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707493920235/ad34097b-4dcf-45ea-bd64-6ded09029c8d.png align="center")

As you can see the title for post with postId:13 is updated.

▪︎ You can query a post using postId as well!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707494144960/67807821-91bf-480f-9f73-7cd6fd3a09a8.png align="center")

▪︎ Let's add a comment on post with postId:13

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707494187161/1e639043-604f-41c5-8551-d8483b3a8036.png align="center")

▪︎ Let's add another comment on post with postId:13

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707494215424/cf4f3ebc-df45-42d8-bba2-5f517e9a124e.png align="center")

▪︎ query all posts

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707494258699/781bfb6d-0e9b-47fa-bafa-b21a72ee9550.png align="center")

You can see 2 comment on post with postId:13

▪︎ You can update, delete the comments as well!

## **4\. Conclusion**

Building a blogging GraphQL API with Django and Strawberry is a straightforward process that provides a flexible and efficient solution for creating and managing blog content. With MyBlogProject as a starting point, you can extend and customize the functionality to suit your specific requirements.