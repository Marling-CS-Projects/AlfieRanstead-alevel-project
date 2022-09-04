# 2.2.10 Cycle 10 - Basic Inter-Nodal Communication

* STOP NODES FROM JUST SIGNING ALL DATA SENT THEIR WAY -> MASSIVE SECURITY FLAW
* Convert pong route from a get request to a post request
* move pong request data from query to body
* create an info getter that just asks a node for all it's info in more detail.

## Design

### Objectives

The objective for this cycle is to turn the [basic inter-nodal communication from Cycle 6](2.2.6-cycle-6-setting-up-inter-nodal-communication.md) into something a little more advanced, allowing Nodes to start using post requests rather than get requests so that the data can be supplied within the body of the request, and other similar quality of life upgrades.

Alongside this there is also a very bad security flaw with the way in which Nodes sign data sent to their pong routes, and that is that they will just sign anything sent their way. This means that node A could just ask node B to sign a transaction that gives all of B's assets to A and then B would just do it and send the signed transaction back to A. This is obviously bad and needs to be fixed.

* [x] Make nodes check the data sent to their pong route and only sign it if it is a dateTime string.
* [x] Convert pong request from a get request to a post request.
* [x] Move pong request data from the query of the request to the body.
* [x] Convert ping/pong terminology into a "handshake" with sufficient renaming

### Usability Features

The main usability feature introduced within this cycle is the renaming and structuring of the "ping/pong" functions and routes within the node software to a "handshake" route which more accurately describes what is actually going on.

### Key Variables

| Variable Name | Use |
| ------------- | --- |
|               |     |
|               |     |
|               |     |

### Pseudocode

#### Only signing date-time objects

Luckily implementing this, and dramatically improving the node software's current security, should actually be pretty easy. All the code will need to do is attempt to parse the message received within the handshake (ping/pong) request as a date-time object and if it succeeds, then the message is okay to be signed and if not then fail the request and do not sign it.

Parsing date-time objects is different for every language and standard library so for this pseudocode the function `Parse_Date` will represent a function which takes a string in the format `YYYY-MM-DD hh:mm:ss` and either returns a date-time object in some format or errors out.

```
// Pseudocode

FUNCTION Valid_Message(message):
    TRY:
        date = Parse_Date(message)
        return date
        
    CATCH ERROR:
        return false
        
    END TRY  
END FUNCTION
```

### Converting the handshake route from a get request to a post request

As both the conversion from a get request to a post request and converting from using queries to using the body to send data is so similar I'll use the below examples to summarise both changes.

#### **Api Route**

Because this is based upon code from a previous cycle where I introduced the nodal communication, I've copied that pseudocode as the base and then edited it from there. [Visit here to see the original Pseudocode](2.2.6-cycle-6-setting-up-inter-nodal-communication.md#the-api-route)

```
// Pseudocode

// import functions from modules made in previous cycles
IMPORT configuration
IMPORT cryptography


// In this case request has changed to be the body of the http request
HTTP_POST_ROUTE handshake (request):

	TRY:
		req_parsed = json.decode(request)
	CATCH ERROR:
		OUTPUT "Incorrect data supplied to handshake"
		RETURN HTTP.code(403) 	
		// a code 403 means "forbidden" in http terms
	END TRY



	OUTPUT "Received handshake request from node claiming to have the public key"
		 + req_parsed.initiator_key

	// using the function defined earlier in this cycle
	time = Valid_Message(req_parsed.message)
	
	IF time == False:
		OUTPUT "Incorrect time format supplied to handshake by node claiming to be"
			+ req_parsed.initiator_key
			
		// this is where a negative grudge would then be stored but that's for a
		// future cycle.
		
		RETURN HTTP.code(403) 
	ENDIF
	
	// time was okay, so store a slight positive grudge
	OUTPUT "Time parsed correctly as:" + time
	
	// how the keys are used has also changed due to the node refactor.
	config = configuration.get_config()
	keys = cryptography.get_keys(config.key_path)

	// create an object that represents the response
	response = {
		responder_key: self.pub_key
		initiator_key: req_parsed.initiator_key
		message: req_parsed.message
		signature: keys.sign(message)
	}
	
	// encode the response to be http safe
	data = json.encode(response)
	
	// return it to the requester
	OUTPUT "Handshake Analysis Complete. Sending response..."
	return HTTP.text(data)
END HTTP_ROUTE

```

#### Requester Function

```
// Some code
```

## Development

\---

### Outcome

### Signing Date-Time Objects

Converting the pseudocode to actual code for this was pretty simple, although I ended up keeping the parsing within the main handshake route function rather than splitting it out into a separate function, so that's why the code isn't in a function below.

```v
// V code - within the "pong" route in /src/modules/server/main.v 

// with this version of the node software all messages should be time objects
time := time.parse(req_parsed.message) or {
    eprintln("Incorrect time format supplied to /pong/:req by node claiming to be $req_parsed.ping_key")
    return app.server_error(403)
}

// time was okay, so store a slight positive grudge
println("Time parsed correctly as: $time")
```

This code can be found in [this commit here](https://github.com/AlfieRan/MonoChain/blob/49d9f53c2253b742795afc40651ba4da988819cb/packages/node/src/modules/server/handshake.v), although bear in mind that the code above removed some in-code comments meant for future use in a different cycle that is not in development and is just being planned at the moment.

### Converting the handshake route from a get request to a post request

#### The Api route

```v
// V code - within the "handshake" route in /src/modules/server/handshake.v 

['/handshake'; post]
pub fn (mut app App) handshake_route() vweb.Result {
	body := app.req.data

	req_parsed := json.decode(HandshakeRequest, body) or {
		eprintln("Incorrect data supplied to /handshake/")
		return app.server_error(403)
	}

	println("Received handshake request from node claiming to have the public key: $req_parsed.initiator_key")


	// THIS IS THE SECTION WRITTEN UNDER THE TITLE "Signing Date-Time Objects"
	// with this version of the node software all messages should be time objects
	time := time.parse(req_parsed.message) or {
		eprintln("Incorrect time format supplied to handshake by node claiming to be $req_parsed.initiator_key")
		return app.server_error(403)
	}

	// time was okay, so store a slight positive grudge
	println("Time parsed correctly as: $time")
	// THIS IS THE SECTION WRITTEN UNDER THE TITLE "Signing Date-Time Objects"


	config := configuration.get_config()
	keys := cryptography.get_keys(config.key_path)

	res := HandshakeResponse{
		responder_key: keys.pub_key
		initiator_key: req_parsed.initiator_key
		message: req_parsed.message
		signature: keys.sign(req_parsed.message.bytes())
	}

	data := json.encode(res)
	println("Handshake Analysis Complete. Sending response...")
	return app.text(data)
}
```

[This version of the code can be found here.](https://github.com/AlfieRan/MonoChain/blob/49d9f53c2253b742795afc40651ba4da988819cb/packages/node/src/modules/server/handshake.v)

### Challenges

Challenges faced in either/both objectives

## Testing

### Tests

| Test | Instructions                                                                    | What I expect                                                                        | What actually happens | Pass/Fail |
| ---- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | --------------------- | --------- |
| 1    | Completing a correct and valid handshake using a time object as the message.    | A series of console logs on both nodes confirming that the handshake was successful. | As expected           | Pass      |
| 2    | Completing an invalid handshake using the string "invalid data" as the message. | For both nodes to record the handshake as unsuccessful and log such to the console.  |                       |           |
| 3    |                                                                                 |                                                                                      |                       |           |

### Evidence

#### Test 1 Evidence - Valid data in a valid handshake

<figure><img src="../.gitbook/assets/image (1) (2).png" alt=""><figcaption><p>The handshake initiator</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>The handshake Recipient</p></figcaption></figure>

As shown, both the initiator and recipient successfully agreed on the handshake, with both showing the same time message to prove this was the same handshake as the test.

#### Test 2 Evidence - Invalid data in the handshake

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption><p>The handshake initiator</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p>The handshake Recipient</p></figcaption></figure>

As shown both nodes classified the handshake as failed, with the initiator predicting that incorrect data may have been sent (although that is not guaranteed as another error may have occurred but in this case we know it was the case of invalid data); and the Recipient logging an incorrect time format supplied and who it was claimed to be supplied by.