# Usage Examples

This document provides sanitized examples of how to use the API client. These examples show the general patterns and capabilities without exposing sensitive implementation details.

## Basic Authentication

```python
from grindr_client import GrindrClient, AccountCredentials

# Initialize the client
client = GrindrClient()

# Create credentials
credentials = AccountCredentials(
    username="your_username",
    password="your_password"
)

# Login
success, message = client.login(credentials)
if success:
    print("Successfully logged in")
else:
    print(f"Login failed: {message}")
```

## Profile Discovery

```python
from grindr_client import Location

# Set your location
location = Location(
    latitude=40.7128,
    longitude=-74.0060,
    city="New York",
    country="United States"
)
client.set_location(location)

# Discover nearby profiles
profiles = client.get_nearby_profiles(
    distance_limit=5000,  # 5km radius
    limit=100
)

# Process results
for profile in profiles:
    print(f"Profile: {profile.display_name}")
    print(f"Distance: {profile.approximate_distance}")
    print(f"Age: {profile.age}")
    print(f"Online: {profile.online_until}")
    print("---")
```

## Messaging

```python
# Send a direct message
client.send_message(
    target_id=123456789,
    text="Hello! How are you doing?"
)

# Send typing indicator
client.send_typing_indicator(target_id=123456789, is_typing=True)

# Simulate typing delay
import time
time.sleep(2)

# Stop typing
client.send_typing_indicator(target_id=123456789, is_typing=False)

# Send another message
client.send_message(
    target_id=123456789,
    text="I'm interested in chatting!"
)

# Get conversation history
messages = client.get_messages_by_ids(
    conversation_id="conv_123456",
    limit=50
)

for msg in messages:
    print(f"From {msg.sender_id}: {msg.text}")
    print(f"Sent at: {msg.timestamp}")
```

## Bulk Operations

```python
# Send messages to multiple recipients
recipient_ids = [123456, 789012, 345678]
message_text = "Check out this feature!"

result = client.bulk_send_messages(
    recipient_ids=recipient_ids,
    text=message_text
)

print(f"Successful: {len(result.successful)}")
print(f"Failed: {len(result.failed)}")
for error_id, error_msg in result.errors.items():
    print(f"Error for {error_id}: {error_msg}")
```

## Profile Management

```python
# Get your profile
my_profile = client.get_my_profile()
print(f"Display Name: {my_profile.display_name}")
print(f"Bio: {my_profile.about_me}")
print(f"Age: {my_profile.age}")

# Update profile information
client.update_profile(
    display_name="New Name",
    about_me="Updated bio text",
    age=28,
    height_cm=180,
    weight_grams=75000
)

# Get profile images
images = client.get_profile_images()
for img in images:
    print(f"Image: {img.media_hash}")
    print(f"State: {img.state}")
    print(f"URL: {img.full_url}")

# Set primary image
client.set_primary_image(image_hash="abc123def456")
```

## Interaction Tracking

```python
# Send a tap
client.send_tap(
    target_id=123456789,
    tap_type=1  # 1 = flame/like
)

# Get received taps
received_taps = client.get_received_taps()
for tap in received_taps:
    print(f"Tap from {tap.profile_id}")
    print(f"Type: {tap.tap_type}")
    print(f"Sent at: {tap.timestamp}")

# Get sent taps
sent_taps = client.get_sent_taps()
for tap in sent_taps:
    print(f"Tap to {tap.receiver_id}")
    print(f"Status: {'Read' if tap.read_on else 'Unread'}")

# Get profile views
views = client.get_profile_views()
print(f"Total views: {views.viewed_count}")
print(f"Most recent viewer: {views.most_recent_profile_id}")
```

## Favorites Management

```python
# Add to favorites
client.add_to_favorites(profile_id=123456789)

# Remove from favorites
client.remove_from_favorites(profile_id=123456789)

# Get nearby profiles (includes favorite status)
profiles = client.get_nearby_profiles()
for profile in profiles:
    if profile.is_favorite:
        print(f"{profile.display_name} is in your favorites")
```

## Account Status & Health

```python
# Check login status
is_logged_in, status = client.check_login_status()
print(f"Logged in: {is_logged_in}")
print(f"Status: {status}")

# Check if account is banned
if client.is_banned():
    print("Account is banned")
else:
    print("Account is active")

# Detect shadow ban
shadow_ban_detected = client.detect_shadow_ban()
if shadow_ban_detected:
    print("Shadow ban detected - visibility may be limited")

# Refresh authentication token
success = client.refresh_auth_token()
if success:
    print("Token refreshed successfully")
```

## Proxy Management

```python
# Set proxy list
proxies = [
    "http://proxy1.com:8080",
    "http://proxy2.com:8080",
    "http://proxy3.com:8080"
]
client.set_proxies(proxies)

# Get current proxy
current_proxy = client.get_current_proxy()
print(f"Using proxy: {current_proxy.url}")

# Rotate to next proxy
client.rotate_proxy()

# Get proxy statistics
stats = client.get_proxy_stats()
print(f"Total proxies: {stats['total']}")
print(f"Working: {stats['working']}")
print(f"Failed: {stats['failed']}")
```

## Conversation Management

```python
# Get all conversations
conversations = client.get_conversations()

for conv in conversations:
    print(f"Conversation: {conv.name}")
    print(f"Participants: {len(conv.participants)}")
    print(f"Unread messages: {conv.unread_count}")
    print(f"Last activity: {conv.last_activity_timestamp}")
    
    # Get preview
    preview = conv.preview
    print(f"Last message: {preview.text}")
    print("---")
```

## Advanced: Custom Message Handlers

```python
# Register custom handlers for specific message types
def on_message_received(message):
    print(f"New message from {message.sender_id}: {message.text}")

def on_typing(message):
    print(f"User {message.sender_id} is typing...")

def on_connection_established(message):
    print("WebSocket connection established")

# Register handlers
client.register_message_callback("message_received", on_message_received)
client.register_message_callback("typing", on_typing)
client.register_message_callback("connection_established", on_connection_established)

# Keep connection alive
import time
try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    client.logout()
```

## Error Handling

```python
from grindr_client import GrindrException, AuthenticationError, RateLimitError

try:
    client.send_message(target_id=123456, text="Hello")
except AuthenticationError:
    print("Authentication failed - token may be expired")
    client.refresh_auth_token()
except RateLimitError:
    print("Rate limit exceeded - waiting before retry")
    import time
    time.sleep(60)
except GrindrException as e:
    print(f"API error: {e}")
```

## Performance Tips

### Batch Operations
```python
# Instead of sending messages one by one
for recipient in recipients:
    client.send_message(recipient, "Hello")  # Slow

# Use bulk operations
client.bulk_send_messages(recipients, "Hello")  # Fast
```

### Connection Reuse
```python
# Create one client and reuse it
client = GrindrClient()
client.login(credentials)

# Perform multiple operations
for i in range(100):
    client.send_message(target_id, f"Message {i}")

# Don't create new clients for each operation
```

### Rate Limiting
```python
# Respect rate limits to maintain account health
import time

for profile in profiles:
    client.send_message(profile.id, "Hello")
    time.sleep(1)  # 1 second delay between messages
```

## Best Practices

1. **Always handle exceptions** - Network issues and API errors can occur
2. **Use rate limiting** - Respect platform limits to avoid bans
3. **Reuse connections** - Create one client and reuse it
4. **Monitor account health** - Regularly check for bans and shadow bans
5. **Use proxies** - Rotate proxies for large-scale operations
6. **Log operations** - Track what your code is doing for debugging
7. **Implement backoff** - Use exponential backoff for retries
8. **Validate input** - Check data before sending to the API

---

For more information, see the main [README.md](README.md) and [FEATURES.md](FEATURES.md).

For inquiries about commercial use or custom implementations, contact us on Telegram: [@YourTelegramHandle](https://t.me/YourTelegramHandle)
