## Week 4 - Pipeline Presentation & Multiplayer State Synchronization

### Part 1: Tool Presentation and Handover

A core responsibility of a technical production role is not just building tools, but ensuring the team understands how to use them. I began Week 4 by delivering a formal presentation to the rest of the class, demonstrating the Discord-to-GitHub asset ingestion bot I had engineered over the previous weeks. 

The presentation covered the pipeline's architecture, its integration with Git LFS, and a live demonstration of how artists can use it to bypass complex Git desktop clients. Communicating technical workflows to non-technical disciplines is a critical soft skill in agile development environments, ensuring the tool actually reduces friction rather than adding to it.

![alt text](image-8.png)
*Figure 5. Slide from my pipeline presentation outlining the data flow between Discord, the local staging environment, and the GitHub repository.*

![alt text](image-9.png)
*Figure 6. Slide demonstrating the user-facing output and error handling designed for the art team.*

(yes this is actually the presentation I gave in class lol)

### Part 2: Multiplayer Replication - The Dealer System

Transitioning back to active engine development, my primary technical objective was syncing the core gameplay loop across the network. A classmate had previously authored a foundational version of the `BP_Dealer` blueprint. However, it was built for local execution. In a multiplayer card game, network authority is paramount; if clients handle their own card dealing, the game is immediately vulnerable to cheating and desynchronization *(Ruiz, 2017)*.

I took ownership of `BP_Dealer` and overhauled its architecture to operate on a strict Client-Server model using Unreal's Remote Procedure Calls (RPCs). 

**Technical Refactoring:**
* **Server Authority:** The array representing the deck of cards exists strictly on the server. Clients do not know the order of the deck.
* **Custom Events (RPCs):** I implemented "Run on Server" custom events. When a player presses a button to draw or interact, their Player Controller sends an RPC to the server. The server executes the logic (e.g., popping an array element from the deck) and assigns the card.
* **Multicasting Gameflow:** Once the server validates the action, it triggers a "Multicast" custom event, instructing all connected clients to play the visual dealing animation and update their UI simultaneously. This ensures the game state remains identical for every player in the session *(Epic Games, s.d.-a)*.

![alt text](image-10.png)
![alt text](image-11.png)
*Figure 7. The BP_Dealer blueprint showcasing the transition from local logic to Server-Authoritative Custom Events for card distribution.*



---

# BIBLIOGRAPHY

*(In order they appear in the writeup)*

Ruiz, J. M. (2017) *Multiplayer Game Development with Unreal Engine 4*. Birmingham: Packt Publishing.

Epic Games (s.d.-a) *RPCs in Unreal Engine*. At: https://dev.epicgames.com/documentation/en-us/unreal-engine/rpcs-in-unreal-engine (Accessed 17/04/2026).

Epic Games (s.d.-b) *Session Interface in Unreal Engine*. At: https://dev.epicgames.com/documentation/en-us/unreal-engine/session-interface-in-unreal-engine (Accessed 17/04/2026).