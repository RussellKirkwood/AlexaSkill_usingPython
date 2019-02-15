# AlexaSkill_usingPython
Alexa Skill that uses Python 2.7 on AWS Lambda

This Skill just asks a simple question of what Customer data you would like to hear about (and see on panel).
The logic does not actually connect to database but shows you the steps that lead to it.

In Alexa Developer Panel you need to create a Skill. And the Skill needs to have a "CustomerDataIntent" INTENT with a Slot named "Item".
This Slot is used to pass Customer ID in via the intent.


from __future__ import print_function
from random import *

# --------------- Helpers that build all of the responses ----------------------

def build_speechlet_response(title, output, reprompt_text, should_end_session):
    return {
        'outputSpeech': {
            'type': 'PlainText',
            'text': output
        },
        'card': {
            'type': 'Simple',
            'title': "SessionSpeechlet - " + title,
            'content': "SessionSpeechlet - " + output
        },
        'reprompt': {
            'outputSpeech': {
                'type': 'PlainText',
                'text': reprompt_text
            }
        },
        'shouldEndSession': should_end_session
    }


def build_response(session_attributes, speechlet_response):
    return {
        'version': '1.0',
        'sessionAttributes': session_attributes,
        'response': speechlet_response
    }


# --------------- Functions that control the skill's behavior ------------------

def get_welcome_response():
    """ initialize the session    """

    session_attributes = {}
    card_title = "Welcome"
    speech_output = "Please tell me what Client data you would like to hear about. " \
                    "Say tell me about Client Number"
    # If the user either does not reply to the welcome message or says something
    # that is not understood, they will be prompted again with this text.
    reprompt_text = "Please tell me what Client you want to hear about " 
                    
    should_end_session = False
    return build_response(session_attributes, build_speechlet_response(
        card_title, speech_output, reprompt_text, should_end_session))
        
def get_CustomerDataIntent_Response(intent, session):

    item = intent['slots']['item']['value']
    x = randint(1, 1000)  
    
    session_attributes = {}
    session_attributesTemp = {x: item}
    
    # session_attributes = {'customerid': item}
    session_attributes = session.get('attributes', {})
    session_attributes.update(session_attributesTemp)
    
    data = '{This is where you would have looked up Customer data and insert here}'
    
    card_title = "Welcome"
    speech_output = "Okay Customer ID " + item + " data is " + data
    reprompt_text = ""
                    
    should_end_session = False
    return build_response(session_attributes, build_speechlet_response(
        card_title, speech_output, reprompt_text, should_end_session))        

        
def handle_session_end_request():
    card_title = "Session Ended"
    speech_output = "Thank you for using Customer Data Lookup Alexa. " \
                    "Have a nice day! "
    # Setting this to true ends the session and exits the skill.
    should_end_session = True
    return build_response({}, build_speechlet_response(
        card_title, speech_output, None, should_end_session))


# --------------- Events ------------------

def on_session_started(session_started_request, session):
    """ Called when the session starts """

    print("on_session_started requestId=" + session_started_request['requestId']
          + ", sessionId=" + session['sessionId'])


def on_launch(launch_request, session):
    """ Called when the user launches the skill without specifying what they
    want
    """

    print("on_launch requestId=" + launch_request['requestId'] +
          ", sessionId=" + session['sessionId'])
    # Dispatch to your skill's launch
    return get_welcome_response()


def on_intent(intent_request, session):
    """ Called when the user specifies an intent for this skill """
    #return handle_session_end_request()

    print("on_intent requestId=" + intent_request['requestId'] +
          ", sessionId=" + session['sessionId'])

    intent = intent_request['intent']
    intent_name = intent_request['intent']['name']

    # Dispatch to your skill's intent handlers
    if intent_name == "CustomerDataIntent":
        return get_CustomerDataIntent_Response(intent, session)    
    elif intent_name == "AMAZON.FallbackIntent":
        return get_welcome_response()
    elif intent_name == "AMAZON.HelpIntent" or intent_name == "AMAZON.NavigateHomeIntent":
        return get_welcome_response()
    elif intent_name == "AMAZON.CancelIntent" or intent_name == "AMAZON.StopIntent":
        return handle_session_end_request()
    else:
        # raise ValueError("Invalid intent")
        return handle_session_end_request()


def on_session_ended(session_ended_request, session):
    """ Called when the user ends the session.

    Is not called when the skill returns should_end_session=true
    """
    print("on_session_ended requestId=" + session_ended_request['requestId'] +
          ", sessionId=" + session['sessionId'])
    # add cleanup logic here


# --------------- Main handler ------------------

def lambda_handler(event, context):
    """ Route the incoming request based on type (LaunchRequest, IntentRequest,
    etc.) The JSON body of the request is provided in the event parameter.
    """
    print("event.session.application.applicationId=" +
          event['session']['application']['applicationId'])

    """
    Uncomment this if statement and populate with your skill's application ID to
    prevent someone else from configuring a skill that sends requests to this
    function.
    """
    # if (event['session']['application']['applicationId'] !=
    #         "amzn1.echo-sdk-ams.app.[unique-value-here]"):
    #     raise ValueError("Invalid Application ID")

    if event['session']['new']:
        on_session_started({'requestId': event['request']['requestId']},
                           event['session'])

    if event['request']['type'] == "LaunchRequest":
        return on_launch(event['request'], event['session'])
    elif event['request']['type'] == "IntentRequest":
        return on_intent(event['request'], event['session'])
    elif event['request']['type'] == "SessionEndedRequest":
        return on_session_ended(event['request'], event['session'])
