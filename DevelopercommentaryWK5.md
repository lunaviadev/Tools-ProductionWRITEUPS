## Week 5 - API Architecture and Server-Authoritative Gameplay Replication

### Part 1: API Design - Asset Ingestion Microservice

Before fully transitioning my focus back to in-engine development, I completed a formal architectural exercise mandated by the unit brief: designing a public-facing RESTful Web API for our Discord-to-GitHub pipeline tool. 

Currently, the Discord bot operates as a monolithic, localized script. However, in a professional production environment, decoupling the front-end user interface (Discord) from the back-end processing logic (Git LFS operations) is a crucial standard. By designing this as a distinct microservice, the pipeline becomes significantly more robust, scalable, and secure *(Fielding, 2000)*. It also opens the door for future integrations—for example, we could eventually author a custom Unreal Engine plugin that hits this exact same API to push assets to GitHub directly from within the editor, bypassing Discord entirely.

**Endpoint Overview & Architectural Justification:**
* **URL:** `/api/v1/assets/upload`
* **HTTP Verb:** `POST`
* **Description:** This endpoint is designed to be stateless. It receives a JSON payload containing metadata and a direct download URL for an asset. The server then fetches the file, routes it to the correct project folder based on a strict `art_type` enum, and commits it to the remote repository via Git Large File Storage (LFS). By offloading the file processing to a dedicated web service, we prevent the Discord client from timing out or hitting memory limits during massive file uploads.

**Example Request & Response Specifications:**
To document the API's call flow for potential future integration by other developers on the team, I mapped out the minimal JSON payloads required to communicate with this endpoint. 

```JSON
// Example Minimal Request
POST /api/v1/assets/upload HTTP/1.1
Content-Type: application/json
Authorization: Bearer <API_ACCESS_TOKEN>

{
  "author_id": "847293847281",
  "art_type": "3D",
  "commit_message": "Updated hero pig character mesh",
  "file_name": "hero_pig_v2.fbx",
  "file_url": "https://cdn.discordapp.com/attachments/..."
}

// Example Minimal Response (Success)
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "success",
  "message": "Asset successfully committed to assets/dropoff",
  "data": {
    "github_url": "https://github.com/University-for-the-Creative-Arts/Greedy_Piggies/blob/assets/dropoff/DropOff/3D/hero_pig_v2.fbx",
    "commit_hash": "7a8b9c0d1e2f3g4h5i"
  }
}
```

**API Reference & Parameter Documentation:**
* `author_id` **(String, Required)**: The Discord ID or unique user identifier. Crucial for establishing a non-repudiable audit trail in the event a corrupted asset breaks the engine build.
* `art_type` **(String, Required)**: A strict enum dictating the routing path (`"2D"`, `"3D"`, or `"Animation"`). This prevents users from arbitrarily creating unauthorized directories in the repository.
* `commit_message` **(String, Required)**: A semantic description of the asset changes for version tracking.
* `file_name` **(String, Required)**: The target file name including its extension.
* `file_url` **(String, Required)**: An authenticated URL where the microservice can securely fetch the raw bytes.
* **Returns:** A structured JSON Object containing a `status` string and a `data` object with the `github_url` and `commit_hash` for immediate user feedback.
* **Failure Cases Handled:** * `400 Bad Request`: Fired if an invalid enum is provided.
    * `401 Unauthorized`: Fired if the Bearer token in the header is missing or invalid, ensuring only authorized team members can push to the repository.
    * `413 Payload Too Large`: Fired if the target file exceeds the server's designated disk-cache capacity.

---



# BIBLIOGRAPHY

*(In order they appear in the writeup)*

Fielding, R. T. (2000) *Architectural Styles and the Design of Network-based Software Architectures*. Ph.D. Dissertation. University of California, Irvine.

Ruiz, J. M. (2017) *Multiplayer Game Development with Unreal Engine 4*. Birmingham: Packt Publishing.

Epic Games (s.d.) *Property Replication in Unreal Engine*. At: https://dev.epicgames.com/documentation/en-us/unreal-engine/property-replication-in-unreal-engine (Accessed 17/04/2026).

Glazer, J. and Madhav, S. (2015) *Multiplayer Game Programming: Architecting Networked Games*. Boston: Addison-Wesley Professional.