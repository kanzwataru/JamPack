## EventBus Usage
EventBus is a static class which routes messages/events to listeners. You can add a listener to receive events and also send out events. Event messages are just a simple struct with fields. Implementation-wise it uses the UnityEvent system to hold essentially "function pointers" to objects' methods that it then calls the moment events are sent.

### Defining Event Messages
Simply define some structs holding the information that needs to be passed. They can also be blank.
```c#
// an example of a damage event,
// that a weapon would send out
struct DamageEvent {
	public GameObject damager;
	public GameObject target;
	public int damage;
}

// an example of a health change,
// that a player would send out
struct HealthChangedEvent {
	public int lastHealth;
	public int health;
};

// an example of an event in a racing game,
// that a checkpoint would send out
struct PassCheckpointEvent {
	public Vector3 position;
	public int points;
}

// an example of an event with no fields
struct PlayerDeadEvent {
}
```
### Sending Events
Use the EventBus's static methods to send out an event, by passing it a struct. If there are any listeners they will get called with the struct passed.
```c#
// send out empty message
EventBus.Emit(new PlayerDeadEvent());

// send out message with filled struct
EventBus.Emit(new PassCheckpointEvent() {
	position = transform.position;
	points = checkpointPoints;
});
```
### Receiving Events
Define a method on the class that takes one argument of the type of message and call `EventBus.AddListener` in `Start`or `Awake`. Take advantage of overloading to have the same method name.
```c#
public class PlayerUI : MonoBehaviour {
	...
	...
	void Start() {
		EventBus.AddListener<PlayerDeadEvent>(HandleEvent);
		EventBus.AddListener<HealthChangedEvent>(HandleEvent);
	}

	void HandleEvent(HealthChangedEvent msg) {
		if(msg.health < msg.lastHealth)
			PlayHurtEffects();
		else
			PlayHealEffects();
		
		SetHearts(msg.health);		
	}	

	void HandleEvent(PlayerDeadEvent msg) {
		DisplayGameOver();
		StartFade();
	}
}
```
### Unsubscribing from Events
To stop receiving events call `EventBus.RemoveListener`
```c#
void HandleEvent(DamageEvent msg) { ... }

void OnDestroy() {
	EventBus.RemoveListener<DamageEvent>(HandleEvent);
}
```
