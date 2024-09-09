Clean up Nexus Repository (Python)

----------------------------------------------------------------------
<pre>
    <code>        
all_items = []

# Establish a session for authentication
session = requests.Session()
session.auth = (USER, PASS)

# 1. List all image names.
# (NOTE: Nexus doesn't directly provide an API to list all image names.
# You might need to fetch all assets and then extract unique image names.)

def fetch_all_items(url, params={}):
    all_items = []
    continuation_token = None
    
    while True:
        if continuation_token:
            params["continuationToken"] = continuation_token
        
        response = session.get(url, params=params)
        data = response.json()
        
        all_items.extend(data.get("items", []))
        
        continuation_token = data.get('continuationToken')
        if not continuation_token:
            break

    return all_items

# 1. List all image names using components (each component represents a Docker image).
all_components = fetch_all_items(f"{NEXUS_URL}/service/rest/v1/components", params={"repository": REPO_NAME})

image_names = {component['name'] for component in all_components}
print(f"Total image names: {len(image_names)}")
print(f"Image names: {image_names}")

# 2. For each image, list its tags.
for image_name in image_names:
    tags = [item['version'] for item in fetch_all_items(f"{NEXUS_URL}/service/rest/v1/search", params={"repository": REPO_NAME, "name": image_name})]

    # 3. Keep only KEEP_COUNT of the most recent tags and identify the rest for deletion.
    tags_to_delete = sorted(tags)[:-KEEP_COUNT]
    
    for tag in tags_to_delete:
        asset_response = session.get(f"{NEXUS_URL}/service/rest/v1/search", params={"repository": REPO_NAME, "name": image_name, "version": tag})
        asset_id = asset_response.json().get('items', [{}])[0].get('id')
        
        print(f"Deleting {image_name}:{tag} with asset ID {asset_id}")
        # if asset_id:
        #     delete_response = session.delete(f"{NEXUS_URL}/service/rest/v1/assets/{asset_id}")
        #     print(f"Deleted {image_name}:{tag} with status code {delete_response.status_code}")

print("Cleanup completed!")
    </code>
</pre>
        
