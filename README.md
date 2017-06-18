# Alexa workshop for GWC

## Step 1: LaunchIntent redirected to one intent

1. Create a skill in developer portal 

 - add one intent 'MyIntent'
 - Configure sample utterance: for example 'say hello'
 
2. Create a Lambda function with alexa-skill-kit-sdk-factskill blueprint and region N. Virginia(us-east-1)

        var Alexa = require('alexa-sdk');

        exports.handler = function(event, context, callback) {
            var alexa = Alexa.handler(event, context);
            alexa.registerHandlers(handlers);
            alexa.execute();
        };
        
        // ADD CONSTANTS HERE

        var handlers = {
            'LaunchRequest': function () {
                this.emit('MyIntent');
            },

            'MyIntent': function () {
                this.emit(':tell', 'Hello World from Alexa!');
            }
            
            // ADD MORE HANDLERS HERE
        };
        
        // ADD HELPER FUNCTIONS HERE
        
See full code: https://github.com/ancutagh/alexa-workshop/wiki/Step-1-LaunchIntent-code

## Step 2: Conversation with Alexa

1. Replace ':tell' with ':ask' in MyIntent

        this.emit(':ask', "Do you want to hear a fact, a joke or plan your trip?", "Fact or joke or plan your trip?");

2. Add "Unhandled" for Alexa to say that she did not have the answer.

          'Unhandled': function() {
                  this.emit(':ask', 'Sorry, I didn\'t get that. Try asking.', 'Try asking.');
          }
        }
        
See full code: https://github.com/ancutagh/alexa-workshop/wiki/Step-2-Begin-conversation
  
  ## Step 3: Add Joke intent for Alexa to respond 
  
  1. Create new intents in developer portal with multiple sample utterances
  
  - JokeIntent  
  - FactIntent
  
  2. Add new intents handlers
  
           /*
           Alexa, ask <invocation_name> for a fact
           Alexa, ask <invocation_name> to tell me a fact
           */
          'FactIntent' : function () {
                this.emit(':tell', 'Singing in the rain');
          },

           /*
            Alexa, ask <invocation_name> for a joke
            Alexa, ask <invocation_name> to tell me a joke
           */
          'JokeIntent' : function () {
                this.emit(':tell', 'Chuck Norris calls David Guetta and asks him: David, what do you mean by the world is mine?');
           },
           
  See full code: https://github.com/ancutagh/alexa-workshop/wiki/Step-3-Joke-and-Fact-intents
  
  ## Step 4: Add random jokes or facts
  
  
  1. Add the helper method
  
 
            function randomPhrase(myData) {
                // the argument is an array [] of words or phrases
                var i = 0;
                i = Math.floor(Math.random() * myData.length);
                return(myData[i]);
            }

  2. Update handlers

         /*
           Alexa, ask <invocation_name> for a fact
           Alexa, ask <invocation_name> to tell me a fact
           */
          'FactIntent' : function () {
                const facts_array = languageStrings.FACTS;
                this.emit(':tell', randomPhrase(facts_array));
          },

           /*
            Alexa, ask <invocation_name> for a joke
            Alexa, ask <invocation_name> to tell me a joke
           */
          'JokeIntent' : function () {
                const jokes_array = languageStrings.JOKES;
                this.emit(':tell', randomPhrase(jokes_array));
           },

3. Add string constants

         const languageStrings = {
                     FACTS: [
                         'A year on Mercury is just 88 days long.',
                         'Fact 2'
                     ],
                     JOKES: [ // jokes from http://www.funology.com/outer-space-jokes/
                         'What is a spaceman’s favorite chocolate?<break time=\"1s\"/> A marsbar!',
                         'Joke 2'
                     ],
                     TRIP_INTRO: ["Let's go travelling!"],
                     HELP_MESSAGE: 'This is a skill for GWC... What can I help you with?',
                     HELP_REPROMPT: 'What can I help you with?',
                     STOP_MESSAGE: 'Goodbye!',
                     LAUNCH_MESSAGE: 'Welcome. I know facts and I even know a joke.',
                     LAUNCH_MESSAGE_REPROMPT: 'Try asking me to tell you something about world.'
         };
         
See full code: https://github.com/ancutagh/alexa-workshop/wiki/Step-4-Random-jokes-and-factsc
         
## Step 5: build your own intent

## Step 6: Plan my trip: fromCity, toCity, travelDate, activity, travelMode

     /*
        Alexa, ask <invocation_name> to plan my trip
        OR
        Alexa, ask <invocation_name> to have a conversation
            Plan my trip
        OR
        Alexa, open <invocation_name>
        Alexa: Do you want to hear ... ?
        => Plan my trip
     */
       'PlanMyTrip': function () {
            //delegate to Alexa to collect all the required slot values
            delegateSlotCollection.call(this);

            // define the intro of the speechOutput
            const tripIntro_array = languageStrings.TRIP_INTRO;
            var speechOutput = randomPhrase(tripIntro_array);

            //compose speechOutput that simply reads all the collected slot values

            //travelMode is optional so we'll add it to the output
            //only when we have a valid activity
            var travelMode = isSlotValid(this.event.request, "travelMode");
            if (travelMode) {
              speechOutput += travelMode;
            } else {
              speechOutput += "You'll go ";
            }

            //Now let's recap the trip
            var fromCity=this.event.request.intent.slots.fromCity.value;
            var toCity=this.event.request.intent.slots.toCity.value;
            var travelDate=this.event.request.intent.slots.travelDate.value;

            speechOutput+= " from "+ fromCity + " to "+ toCity+" on "+travelDate;

            //activity is optional so we'll add it to the output
            //only when we have a valid activity
            var activity = isSlotValid(this.event.request, "activity");
            if (activity) {
              speechOutput += " to go "+ activity;
            }

            //say the results
            this.emit(":tell",speechOutput);
        },

        // Helper functions

        function delegateSlotCollection(){
      console.log("in delegateSlotCollection");
      console.log("current dialogState: "+this.event.request.dialogState);
        if (this.event.request.dialogState === "STARTED") {
          console.log("in Beginning");
          var updatedIntent=this.event.request.intent;
          //optionally pre-fill slots: update the intent object with slot values for which
          //you have defaults, then return Dialog.Delegate with this updated intent
          // in the updatedIntent property
          this.emit(":delegate", updatedIntent);
        } else if (this.event.request.dialogState !== "COMPLETED") {
          console.log("in not completed");
          // return a Dialog.Delegate directive with no updatedIntent property.
          this.emit(":delegate");
        } else {
          console.log("in completed");
          console.log("returning: "+ JSON.stringify(this.event.request.intent));
          // Dialog is now complete and all required slots should be filled,
          // so call your normal intent handler.
          return this.event.request.intent;
        }
    }

    function isSlotValid(request, slotName){
        var slot = request.intent.slots[slotName];
        //console.log("request = "+JSON.stringify(request)); //uncomment if you want to see the request
        var slotValue;

        //if we have a slot, get the text and store it into speechOutput
        if (slot && slot.value) {
            //we have a value in the slot
            slotValue = slot.value.toLowerCase();
            return slotValue;
        } else {
            //we didn't get a value in the slot.
            return false;
        }

## Last step: cancel, stop, session ended intents

     'AMAZON.HelpIntent': function () {
          const speechOutput = languageStrings.HELP_MESSAGE;
          const reprompt = languageStrings.HELP_MESSAGE;
          this.emit(':ask', speechOutput, reprompt);
      },
      'AMAZON.CancelIntent': function () {
          this.emit(':tell', languageStrings.STOP_MESSAGE);
      },
      'AMAZON.StopIntent': function () {
          this.emit(':tell', languageStrings.STOP_MESSAGE);
      },
      'SessionEndedRequest': function () {
          this.emit(':tell', languageStrings.STOP_MESSAGE);
      },
