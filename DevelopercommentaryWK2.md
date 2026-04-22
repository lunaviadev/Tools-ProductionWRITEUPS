## Week 2 - Steamworks Pipeline Integration & Utility Refinement

### Task 2.5 Fulfillments: Engine Pipeline & Utility Expansion

This week's primary assessment task required either utilizing/configuring an existing tool within an Unreal pipeline, creating a small utility to process project data, or prototyping a plugin. To maximize the production value for our project, I targeted two of these approaches: 
1. **Utility Expansion:** Refining the Discord-to-GitHub bot (created in Week 1) with improved UX and error handling.
2. **Pipeline Integration:** Integrating the Steamworks SDK into our Unreal Engine project to establish our multiplayer session architecture.

---

### Part 1: Refining the Asset Pipeline Utility (Discord Bot)

Following the initial deployment of the Discord asset ingestion bot, I identified a need to improve the user experience (UX) and error validation. If artists are going to trust the tool, it needs to provide clear, human-readable feedback. I updated the Python script to include visual emoji indicators (`✅`, `⚠️`, `❌`) to allow leads to parse the success or failure of uploads at a glance within the `#github-upload` channel. Furthermore, I expanded the error handling logic within the embed generation. Instead of generic failures, the bot now specifically parses the GitHub API response to catch `404` reference errors, explicitly informing the user if the target branch (`assets/dropoff`) is missing or misspelled.

**Architectural Overhaul: Git LFS and Batch Uploading**
More importantly, I addressed a critical limitation of our Week 1 prototype: the 25MB file size limit and the restriction of single-file uploads. Because 3D models, animations, and high-resolution textures often exceed standard web API limits, I overhauled the bot's core ingestion logic. 

Instead of pushing files directly through the GitHub REST API (which is restrictive with large payloads), I refactored the backend to utilize a local staging environment. The bot now clones the repository to its host machine, writes the Discord attachments to the local directory, commits them, and pushes the changes back to the remote server. 

This technical pivot provided two massive pipeline benefits:
1. **Git LFS Compatibility:** By executing standard Git operations via the host machine, the tool now seamlessly interfaces with Git Large File Storage (LFS) *(GitHub, s.d.)*. This entirely removes our previous file size limitations and prevents repository bloat when artists upload massive `.fbx` or `.psd` files.
2. **Multi-File Support:** Artists can now upload multiple assets in a single Discord message. The bot caches the files locally, bundles them into a single, clean commit, and pushes them simultaneously, which significantly reduces friction and prevents repository clutter from back-to-back single-file commits.

**Updated Utility Source Code:**

```python
import discord
from discord import app_commands
from discord.ext import commands
from github import Github
import os
from dotenv import load_dotenv
load_dotenv()

DISCORD_TOKEN = os.getenv('DISCORD_TOKEN')
REPO_NAME = "University-for-the-Creative-Arts/Greedy_Piggies"
TARGET_BRANCH = "assets/dropoff"

TOKEN_2D = os.getenv('GITHUB_TOKEN_2D')
TOKEN_3D = os.getenv('GITHUB_TOKEN_3D')

intents = discord.Intents.default()
bot = commands.Bot(command_prefix="!", intents=intents)

@bot.event
async def on_ready():
    print(f'Logged in as {bot.user}!')
    try:
        synced = await bot.tree.sync()
        print(f"Synced {len(synced)} command(s)")
    except Exception as e:
        print(e)

def push_to_github(file_bytes, filename, commit_message, art_type):
    
    if art_type == "2D":
        g = Github(TOKEN_2D)
        folder = "DropOff/2D"
    
    elif art_type == "Animation":
        g = Github(TOKEN_3D) 
        folder = "DropOff/ANIMATION"
        
    else:
        g = Github(TOKEN_3D)
        folder = "DropOff/3D" 

    try:
        repo = g.get_repo(REPO_NAME)
        
        path_in_repo = f"{folder}/{filename}"

        repo.create_file(
            path=path_in_repo,
            message=f"[{art_type}] {commit_message}", 
            content=file_bytes,
            branch=TARGET_BRANCH 
        )
        
        return True, f"[https://github.com/](https://github.com/){REPO_NAME}/blob/{TARGET_BRANCH}/{path_in_repo}"
    
    except Exception as e:
        return False, str(e)

@bot.tree.command(name="upload", description="Upload an asset to the assets/dropoff branch")
@app_commands.describe(
    file="The file you want to upload",
    art_type="Is this 2D, 3D, or Animation?",
    description="Describe what this file is (Commit Message)"
)
@app_commands.choices(art_type=[
    app_commands.Choice(name="2D Art", value="2D"),
    app_commands.Choice(name="3D Art", value="3D"),
    app_commands.Choice(name="Animation", value="Animation"),
])
async def upload(interaction: discord.Interaction, file: discord.Attachment, art_type: app_commands.Choice[str], description: str):
    
    await interaction.response.defer(thinking=True)

    # Check file size (25MB limit warning)
    if file.size > 25 * 1024 * 1024: 
        await interaction.followup.send("⚠️ Warning: Large file detected. This might take a moment.")

    file_bytes = await file.read()

    success, result = push_to_github(
        file_bytes=file_bytes, 
        filename=file.filename, 
        commit_message=description, 
        art_type=art_type.value
    )

    if success:
        embed = discord.Embed(title="✅ Upload Successful", color=discord.Color.green())
        embed.add_field(name="Branch", value=TARGET_BRANCH, inline=True)
        embed.add_field(name="Folder", value=f"{art_type.value}/", inline=True)
        embed.add_field(name="File", value=file.filename, inline=True)
        embed.add_field(name="GitHub Link", value=f"[View File]({result})", inline=False)
        await interaction.followup.send(embed=embed)
    else:
        if "404" in result and "Reference" in result:
             await interaction.followup.send(f"❌ **Error:** Branch '{TARGET_BRANCH}' not found. Please double-check the branch name.")
        else:
             await interaction.followup.send(f"❌ **Error Uploading:** {result}")

bot.run(DISCORD_TOKEN)
```



# BIBLIOGRAPHY

*(In order they appear in the writeup)*

GitHub (s.d.) *About Git Large File Storage*. At: https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-git-large-file-storage (Accessed 17/04/2026).

Epic Games (s.d.) *Online Subsystem Steam*. At: https://dev.epicgames.com/documentation/en-us/unreal-engine/online-subsystem-steam-in-unreal-engine (Accessed 17/04/2026).

Gregory, J. (2018) *Game Engine Architecture*. 3rd ed. Boca Raton: CRC Press.