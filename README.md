import os
import datetime
import webbrowser
import speech_recognition as sr
import pyttsx3
import wikipediaapi

# Initialize the speech engine for Text-To-Speech (TTS)
try:
    engine = pyttsx3.init()
    # Optional: Adjust speech rate (speed) and volume
    engine.setProperty('rate', 175)
    engine.setProperty('volume', 1.0)
except Exception as e:
    print(f"Error initializing TTS engine: {e}")

def speak(text):
    """Makes the assistant speak out loud."""
    print(f"Assistant: {text}")
    engine.say(text)
    engine.runAndWait()

def greet_user():
    """Greets the user based on the current time of day."""
    hour = datetime.datetime.now().hour
    if 0 <= hour < 12:
        speak("Good morning!")
    elif 12 <= hour < 18:
        speak("Good afternoon!")
    else:
        speak("Good evening!")
    speak("I am your assistant. How can I help you today?")

def listen_command():
    """Listens to microphone input and converts speech to text."""
    recognizer = sr.Recognizer()
    
    with sr.Microphone() as source:
        print("\nListening...")
        # Adjust for background noise to minimize ambient misinterpretation
        recognizer.adjust_for_ambient_noise(source, duration=0.5)
        recognizer.pause_threshold = 0.8
        
        try:
            audio = recognizer.listen(source, timeout=5, phrase_time_limit=5)
            print("Recognizing...")
            # Use Google Speech Recognition API (Requires internet)
            query = recognizer.recognize_google(audio, language='en-in')
            print(f"User said: {query}\n")
            return query.lower()
        except sr.WaitTimeoutError:
            return "none"
        except sr.UnknownValueError:
            print("Could not understand the audio.")
            return "none"
        except sr.RequestError:
            speak("Network error. Please check your internet connection.")
            return "none"
        except Exception as e:
            print(f"An unexpected error occurred: {e}")
            return "none"

def execute_assistant():
    """Core logic loop processing user requests."""
    greet_user()
    
    # User agent format required to avoid Wikipedia block errors
    wiki = wikipediaapi.Wikipedia(
        user_agent='MyVoiceAssistant/1.0 (contact@example.com)', 
        language='en'
    )
    
    while True:
        query = listen_command()
        
        if query == "none":
            continue
            
        # Command 1: Search Wikipedia
        if 'wikipedia' in query:
            speak('Searching Wikipedia...')
            query = query.replace("wikipedia", "").strip()
            page = wiki.page(query)
            
            if page.exists():
                # Provide a snapshot summary of the article
                summary = page.summary[0:200]
                speak("According to Wikipedia...")
                speak(summary)
            else:
                speak(f"Sorry, I couldn't find a Wikipedia page for {query}.")

        # Command 2: Open YouTube
        elif 'open youtube' in query:
            speak("Opening YouTube.")
            webbrowser.open("https://www.youtube.com")

        # Command 3: Open Google
        elif 'open google' in query:
            speak("Opening Google.")
            webbrowser.open("https://google.com")

        # Command 4: Check current time
        elif 'the time' in query or 'current time' in query:
            str_time = datetime.datetime.now().strftime("%H:%M:%S")
            speak(f"The current time is {str_time}")

        # Command 5: Safely terminate the assistant loop
        elif 'exit' in query or 'stop' in query or 'bye' in query:
            speak("Goodbye! Have a productive day.")
            break
            
        else:
            speak("Command not recognized. Try saying 'open google', 'the time', or 'exit'.")

if __name__ == "__main__":
    execute_assistant()
    
