## CHAPTER 12 - Design a Chat System

### STEP 1: Understand the problem and establish a design scope

1. Validate what type of app we are designing, whether it's a web app/ a mobile app/ both? both 
2. Who does it serve as, either a one-on-one chat app/ a group chat? both
3. How many daily active users? 50M
4. Are we designing it for a massive scale or a startup? 
5. What type of messages will the users be able to send: text, attachments? only text
6.  What should be the limit of the characters in each message? 100,000 chars
7.  The maximum number of people who can join the group?
8.  How long do we retain chat history?
9.  important features - one-on-one chat, group chat, showing online presence, and sending text messages.

### Step 2: Propose high-level design and get a buy-in

    In a chat application, clients don't communicate with each other directly; 
    Instead, there will be a chat service that acts as a medium between clients. 
        - which will find the right recipient and transfer the message,
        - if the user isn't online, it holds the message and sends it to them once they're online
        - receive messages from other clients.


    Requests are initiated at the client side in client-server systems, so for sending a message to the chat server,
    The client can use the HTTP time-tested protocol to send the message with a keep-alive header in the message to 
    maintain a persistent connection with the chat service.

    But on the receiver side, it's a bit more complicated to send messages from the server. Many techniques are used to 
    simulate server-initiated connections using polling, websockets, and long polling.

    * Polling: 
        In this technique, the client periodically asks the client if there are any new messages available. If so, just return the messages. 
        Depending on the frequency of our checking if the messages are available wastes a  lot of resources and can be costly.

    * Long Polling: 
        Polling is inefficient; for that reason, we make use of long polling, which opens a connection with the server and
        checks if there are new messages. If there are any the server returns the messages and closes the connection; if not,
        when a timeout is met, then the connection will be closed.
        and periodically, a new connection opens and continues the same steps.

        Long polling could also be inefficient when,
            - When the user doesn't chat much, but the periodic requests are sent to the server, wasting resources. 
            - The server can't tell if a client has disconnected. 
            - The sender and receiver might not connect to the same chat server, because HTTP-based servers are stateless.
            When we use round robin for load balancing, the server we connect to might not have a long polling connection with 
            the client who receives the message.
        
    * Web Sockets: 
        They are used for sending asynchronous updates from the server to the client. websocket connection is initiated by the client and
        makes a 3-way handshake with the acknowledgements. Through this persistent connection in place, WebSockets work even if a firewall 
        is in place because they use http/https port 80/443.

        making it a choice to use both at the sender and the receiver side. Since they are persistent connections, an efficient connection 
        -management is critical on the server side.


    
        
        
        
          
  
        
    
    


