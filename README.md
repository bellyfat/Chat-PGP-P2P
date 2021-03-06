# Chat-PGP-P2P
The goal of this project is to create a P2P chat application secure.
It will use the PGP concept to secure its communications.

It use the open source GnuPG to encrypt/decrypt messages and manage public/secret key.

## 1. Local architecture
The application can be defined on two processes :
  - a core process which manage all interactions with other contacts;
  - an GUI process which display and receive order from the user.
  
![alt tag](https://docs.google.com/drawings/d/1QxrPv2GBXfWVg0UZ9v2EOpYFAVSnTN4NwQdItwjrMog/pub?w=1151&h=592)

### 1.A. The core process
The core process is a background process working like a server.

When this process is launch, it open a UDP listener to add new contacts and a TCP listener to allow discussion requests. 
It open also TCP connections for each new discussion created.

It listen each connections to get new contacts and new messages.
It check also the GUI process for new messages to send or for new discussions to create.
Only non-block sockets is used to allow the core process to listen two or more listeners/connections. 

It memorize each opened TCP connections, each generated/shared key and each messages sended and received from/to other contacts.
That makes a good separation between the GUI interface and the working process.

### 1.B. The GUI process
The GUI process has an interface to communicate with the core process.
A web interface, a command application, or a GUI application could be used independantly to the core process.

It send requests regulary to the core process to keep its views updated.

The GUI process should not contain any data like messages or connections sockets except its own socket to discuss with the core process.
It will only retrieve informations from the core process to print them without memorize them.

## 2. Protocol
Each application have several couples of public/private keys :
 - one in general to begin a discussion and to authentificate every messages from user;
 - one for each discussion to separe each discussion from a same user.

The client and server have many roles.

### 2.A. Who's there (UDP mode)
First of them is to find neighbours arround.
For that, there are two ways:
 - the core process send an UDP request allow it to be seen by other contacts and wait for responses;
 - when the core process receive an UDP request from other contacts.
 
![alt tag](https://docs.google.com/drawings/d/1Ztd4E9MDaGBN20A3rO09nqmKqa9q4oCFvaX7XVc6CbI/pub?w=2020&h=597)
 
For this recognition task, the general couple of keys will be used to authentificate each contact when discussion request is made.
The UDP broadcast message contains the owner public key and the UDP response message contains the distant contact public key.
None of theses messages are crypted.

This step only help the application to be seen by others and is done regulary.

### 2.B. Let's talk together (TCP mode)
After some UDP messages exchanged, a list of contacts should be available and the user can send requests to open discussions.

![alt tag](https://docs.google.com/drawings/d/18N_1828-j1j96WrsqvAtdIF_4W71BCFZOvxV8uAnnlI/pub?w=1014&h=959)

As long as new couples of keys aren't been created in both sides and publics keys aren't been communicated, all messages will be encrypted by using the general couple of keys of each application.

When an user wants to open a conversation to another, it create an TCP tunnel to communicate. 
But before that, a secret password have to be transmitted by other secure transmission way to recognize the distant user behind the application.

Then a couple of public/private key specific to this discussion is created and the user application send an request to the distant application including the public key and the prompted password.

Depending on the secret password received, the distant user has the choice to keep the connection or to refuse it.
If it is the right password, it send its public key created for this discussion. In the other case, it will close the connection.

At the end of this step a discussion between the two users is opened. They can now exchange some messages together.

### 2.C. What's up (TCP mode)
The discussion opened, both users can send signed and crypted messages by using its private key and the distant user's public key.

![alt tag](https://docs.google.com/drawings/d/13NqTWwHt7ozKSw3iij0Wb2Qfr89rIuoA4ILcz4T9pOY/pub?w=1009&h=486)

A discussion between two users is considered as close when one of these users has close the TCP tunnel.
The other user will see this closure when the core process will listen the concerned tunnel or when user would try to send a message.

### 2.D. Big announcement
For some reason, we could imagine that someone wants to open a unique discussion with several users. To do so, two possibilities could easly be used :
 - each user create one discussion per other concerned users and when one user wants to send a grouped messages, it send it using all opened discussions;
 - one user is chosen as the administrator of the discussion and when one user want to send a grouped message, it send it to the admistrator which send it to all other concerned users.
 
Both solutions are quite simple to implement but is not really secure. In the first case, each user can get different list of users and so, a specific user might send a grouped message to a little part of the groupe only. In the second one, all users except the administrator don't really know which users are included in the discussion or not.

Another way is to open an only one discussion for all these users. Instead of opening one discussion for both other users, only one would be created.
To send a grouped message, an user have to encrypt the message by using its own private key and public keys from all other users.
To do so, an specific order between each key should be defined to allow both users to decrypt the sended message. It could be the order of user add in this discussion.

This method force each users to get all other public keys, and so, aware each users which users are concerned by this discussion.
If a user doesn't know which users are in a discussion, it simply cannot send any message using this grouped discussion because some encryption operations are missing.

A continous synchronization should also be made to preserve encryption order especially when a new user is added.
During this step of synchronization, each user could have the choice to allow an user entrance or to refuse it, giving them a perfect control of the grouped discussion.
In addition of this choice, all users have the possibility to suggest a new member to this duscussion agreed or not by others.

## 3. Peer to peer on internet
The main problem of this project is probably to find contacts arround us.
When the application is only used into an internal network, that isn't problematic : we can send a broadcast request like explain at part 2.A.
But when we want to use it outside, this kind of broadcast request cannot be done. Firewalls usually block them.

Even if broadcast requests is impossible to find some neightbors, a UDP connection with others still be possible.
By using a specific IP address, two neighbours can be joined.
Then a discussion could be created between these two users like descript at part 2.B.

That why, we need to find other way to find some neightbors by getting some IP address.

### 3.A. Using a file
One solution could be a list of contacts which contains IP addresses of potential neighbours.
This kind of list is already used by bitorrent applications which use a torrent file to connect to others.

In our case, instead of send one UDP brocast request, the application send that UDP request on each contact from the list.
This list can also be completed with the neighbours list contents to get more contacts.
To do so, when the client send a request to another user, the distant server send a list of its contacts in addition to its public key.

### 3.B. Using a third-party server
Another solution could be a third-party server which contains a regulary updated list of contacts.
This kind of system is already used by applications such as Facebook or Twitter for some functionalities.

In our case, the application can connect to this third-party server to get a list of contacts.
Then it send the UDP request on each contact like explain at part 2.A.
Of course, by receiving the request of contacts, this third-party server should add the requested user IP address to its list of contacts for few times.

The application would send this kind of request regulary to the third-party server for more contacts. By this way, it also aware the server that it still be available to discuss with others.
