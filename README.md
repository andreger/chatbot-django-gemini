# How to Develop an AI Chatbot Application with Python Django and Google Gemini API

This guide equips you with the steps to bring your chatbot to life, empowering you to create engaging user experiences with a powerful AI chatbot built using Django and Google Gemini API.

## About the author - André Gervásio

20+ years of experience in software development. Bachelor's degree in Computer Science. Fullstack Software Engineer and Tech Lead in Java, Python, JS, PHP applications. Certified in Front-End, Back-End and DevOps technologies. Experienced in Scrum and Agile Methodologies. Solid knowledge in Databases and ORMs. Practical skills in Cloud Computing Services.

[Visit my LinkedIn](https://www.linkedin.com/in/andregervasio/)


## 1. Install Python and Django

Before diving into code, ensure you have the following tools:

* Verify Python installation using the following command in your terminal:

```
python3 --version # or python --version
```

* If it is installed, you should see the version number. Otherwise, download it from [https://www.python.org/downloads/](https://www.python.org/downloads/).

* Create a directory for your project. In this article, we'll use the name **chatbot-django-gemini**. Access it and create a virtual environment to isolate project dependencies:

```
mkdir chatbot-django-gemini
cd chatbot-django-gemini
python3 -m venv venv
```

* Activate the virtual environment:

```
source venv/bin/activate
```

* Install Django using pip:

```
pip install django
```

## 2. Create a Django Project

A Django project serves as the foundation for your chatbot application.

* Open your terminal and navigate to your desired project directory.

* Execute the following command to create a new Django project named **chatbot** in the current directory:

```
django-admin startproject chatbot .
```

## 3. Create a Django App

Django apps organize functionalities. We'll create one for our chatbot logic:

* Within your **chatbot** project directory, run this command:

```
python3 manage.py startapp chatbotapp
```

## 4. Integrate Google GenerativeAI Library

The Google GenerativeAI library allows us to interact with the Gemini API for generating responses.

* Use pip to install the library:

```
pip install google-generativeai
```

* An API key is required to use the GenerativeAI library.

* Browse to [https://ai.google/](https://ai.google/) and create or select a project.

* Go to the Get API Key page.

* Click on Create API Key to obtain an API key. Store it securely (we'll use an environment variable later).

## 5. Configure Django Settings

* Now, let's configure Django to work with our chatbot app and API key.

Django projects are organized by individual applications, each focusing on specific functionalities. In step 3, you created a Django app named **chatbotapp** to encapsulate the logic behind your chatbot. Here's why we need to add it to **INSTALLED_APPS**:

  * Modular Project Structure: Django projects promote a modular approach. By creating separate apps, you can organize your codebase efficiently and maintain a clean separation of concerns. Each app can have its own models, views, templates, and other components specific to its functionality.
  * App Recognition: When you add **chatbotapp** to **INSTALLED_APPS**, you essentially tell Django: "Hey, this **chatbotapp** exists, and it's part of this project. Recognize it and include its functionalities when running the project."
  * Component Discovery: Including **chatbotapp** in **INSTALLED_APPS** allows Django to discover the components (models, views, templates) within your **chatbotapp**. These components are crucial for building the chatbot's features.

* Open your project's **chatbot/settings.py** file.

* Inside the **INSTALLED_APPS** list, add **'chatbotapp'**.

* We'll use an environment variable for security. In your terminal, set the API key using a command like:

```
export GENERATIVE_AI_KEY="YOUR_API_KEY_HERE"
```

* Replace **YOUR_API_KEY_HERE** with your actual key.

* In **settings.py**, add the following code to access the key securely:

```python
GENERATIVE_AI_KEY = os.environ.get('GENERATIVE_AI_KEY') # Don't forget to import os package

if not GENERATIVE_AI_KEY:
    raise ValueError('GENERATIVE_AI_KEY environment variable not set')
```

## 6. Define the Chatbot Model

This step allows you to store and manage chat interactions in your database.

* In your **chatbotapp/models.py** file, define a model named **ChatMessage** using Python classes that inherit from Django's **Model** class.

```python
from django.db import models

class ChatMessage(models.Model):
    user_message = models.TextField()
    bot_response = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"User: {self.user_message}, Bot: {self.bot_response}"
```

* If you want to use the Django admin interface to manage chat history, register your model in **chatbotapp/admin.py**:

```python
from django.contrib import admin
from .models import ChatMessage

admin.site.register(ChatMessage)
```

* To apply the model changes to your database, run the following commands in your terminal:

```
python3 manage.py makemigrations
python3 manage.py migrate
```

## 7. Create Views for User Interaction

Now we'll delve into the heart of our chatbot's functionality - handling user interactions and managing conversation history. This is achieved by creating views in Django. Views are like controllers that handle user requests and generate responses.

* Create a view in **chatbotapp/views.py**.

* In our case, we'll define two views:

  1. **send_message:** This view handles user messages submitted through a form. It retrieves the user's message, interacts with the Gemini API and generates a response. Additionally, it saves the conversation and redirects the user to another view displaying the chat messages.  
  2. **list_messages:** This view retrieves all stored chat messages from the database and renders them in a template (which you'll create in a further step). This allows users to see the conversation history and track their interactions with the chatbot.

```python
from django.shortcuts import redirect, render
from chatbot.settings import GENERATIVE_AI_KEY
from chatbotapp.models import ChatMessage
import google.generativeai as genai

def send_message(request):
    if request.method == 'POST':
        genai.configure(api_key=GENERATIVE_AI_KEY)
        model = genai.GenerativeModel("gemini-pro")

        user_message = request.POST.get('user_message')
        bot_response = model.generate_content(user_message)

        ChatMessage.objects.create(user_message=user_message, bot_response=bot_response.text)

    return redirect('list_messages')

def list_messages(request):
    messages = ChatMessage.objects.all()
    return render(request, 'chatbot/list_messages.html', { 'messages': messages })
```

## 8. Configure Routes

In Django, URLs map user requests to specific views. This step involves defining routes within your chatbot app and including them in your main project's URL configuration.

* Create a new file **chatbotapp/urls.py**. This file will define URL patterns specific to your chatbot app.

```python
from django.urls import path
from .views import send_message, list_messages

urlpatterns = [
    path('send', send_message, name='send_message'),
    path('', list_messages, name='list_messages'),
]
```

* Now, you need to include the chatbot app's URL patterns within your main project's URL configuration file **chatbot/urls.py**.

```python
from django.urls import path, include  # Don't forget to import include function
from adminsite import admin

urlpatterns = [
    path('admin/', admin.site.urls),
    path('chatbot/', include('chatbotapp.urls')), # Include the chatbot app's URL patterns
]
```

## 9. Build Templates for User Interface

Templates define the structure and presentation of your chatbot's user interface.

* Create a template in **templates/chatbot/list_messages.html**.

* This template will likely include an input field for the user's message, a button to submit the message, and a space to display the chatbot's response.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Chatbot</title>
</head>
<body>
    {% for message in messages %}
    <div>
        <strong>User:</strong>  {{ message.user_message }}
    </div>
    <div>
        <strong>Bot:</strong>  {{ message.bot_response }}
    </div>
    {% endfor %}

    <form action="{% url 'send_message' %} " method="post">
        {% csrf_token %}
        <textarea name="user_message"></textarea>
        <input type="submit" value="Send">
    </form>
</body>
</html>
```

* To ensure Django recognizes your template directory, you need to update the **TEMPLATES** setting in your **chatbot/settings.py** file.

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],  # Update DIRS path. Don't forget to include os package.
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [],
        },
    },
]
```

## 10. Run the Server and Test Your Chatbot

* In your terminal, navigate to your project directory and start the Django development server using:

```
python3 manage.py runserver
```

* This will typically launch the server on port 8000 by default.

* Open your web browser and visit the chatbot URL defined in your project's URL configuration (http://127.0.0.1:8000/chatbot). This should render your chatbot interface.

## Conclusion

These are the essential steps to build a basic AI chatbot using Django and Gemini API. This provides a solid foundation for crafting interactive and informative chatbots that can engage with users.

You can extend this functionality in various ways:

* **User Authentication:** Implement user authentication to personalize the chatbot experience and potentially access user data for more tailored responses.
* **Advanced Chat functionalities:** Explore integrating features like buttons, images, or links within the chatbot responses for a richer user interaction.
* **Custom GenerativeAI Models:** Consider experimenting with different GenerativeAI models offered by Gemini to find the best fit for your specific use case.

By leveraging the power of Django and Gemini API, you can create versatile AI chatbots that can streamline communication, answer questions, and provide valuable interactions for your users. Explore the possibilities!
