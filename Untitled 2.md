### All data classes representing Events in this project

Below are every Kotlin `data class` that implements an `Event` type (directly or via a specific `*Event` interface), grouped by feature and including their file locations.

#### Domain: Accommodation events

- `AccommodationCreatedEvent` — `domain/.../event/AccommodationEvent.kt`
- `AccommodationBookedEvent` — `domain/.../event/AccommodationEvent.kt`
- `AccommodationBookingCanceledEvent` — `domain/.../event/AccommodationEvent.kt`
- `AccommodationExpiredEvent` — `domain/.../event/AccommodationEvent.kt`

#### Domain: Attraction events

- `AttractionCreatedEvent` — `domain/.../event/AttractionEvent.kt`
- `AttractionBookedEvent` — `domain/.../event/AttractionEvent.kt`
- `AttractionBookingCanceledEvent` — `domain/.../event/AttractionEvent.kt`
- `AttractionExpiredEvent` — `domain/.../event/AttractionEvent.kt`
- `AttractionFullEvent` — `domain/.../event/AttractionEvent.kt`
- `AttractionAvailableEvent` — `domain/.../event/AttractionEvent.kt`

#### Domain: Commute events

- `CommuteCreatedEvent` — `domain/.../event/CommuteEvent.kt`
- `CommuteBookedEvent` — `domain/.../event/CommuteEvent.kt`
- `CommuteBookingCanceledEvent` — `domain/.../event/CommuteEvent.kt`
- `CommuteExpiredEvent` — `domain/.../event/CommuteEvent.kt`
- `CommuteFullEvent` — `domain/.../event/CommuteEvent.kt`
- `CommuteAvailableEvent` — `domain/.../event/CommuteEvent.kt`

#### Domain: Travel offer events

- `TravelOfferCreatedEvent` — `domain/.../event/TravelOfferEvent.kt`
- `TravelOfferReservedEvent` — `domain/.../event/TravelOfferEvent.kt`
- `TravelOfferReservationCanceledEvent` — `domain/.../event/TravelOfferEvent.kt`
- `TravelOfferBookedEvent` — `domain/.../event/TravelOfferEvent.kt`
- `TravelOfferReleaseEvent` — `domain/.../event/TravelOfferEvent.kt`
- `TravelOfferBookingCanceledEvent` — `domain/.../event/TravelOfferEvent.kt`
- `TravelOfferRebookedEvent` — `domain/.../event/TravelOfferEvent.kt`
- `TravelOfferExpiredEvent` — `domain/.../event/TravelOfferEvent.kt`
- `TravelOfferMadeUnavailableEvent` — `domain/.../event/TravelOfferEvent.kt`
- `TravelOfferMadeAvailableEvent` — `domain/.../event/TravelOfferEvent.kt`

#### Domain: Booking events

- `BookingCreatedEvent` — `domain/.../event/BookingEvent.kt`
- `ProcessBookingEvent` — `domain/.../event/BookingEvent.kt`
- `CompleteBookingEvent` — `domain/.../event/BookingEvent.kt`
- `BookingCancelRequestedEvent` — `domain/.../event/BookingEvent.kt`
- `BookingCancelRequestedFailedEvent` — `domain/.../event/BookingEvent.kt`
- `CancelBookingEvent` — `domain/.../event/BookingEvent.kt`
- `ProcessCancelBookingEvent` — `domain/.../event/BookingEvent.kt`
- `FailBookingEvent` — `domain/.../event/BookingEvent.kt`
- `FailCancelBookingEvent` — `domain/.../event/BookingEvent.kt`

#### Application: Compensation events (also implement domain `*Event` interfaces)

- `AccommodationBookedCompensatedEvent` — `application/.../event/CompensationEvents.kt`
- `AccommodationBookingCanceledCompensatedEvent` — `application/.../event/CompensationEvents.kt`
- `AttractionBookedCompensatedEvent` — `application/.../event/CompensationEvents.kt`
- `AttractionBookingCanceledCompensatedEvent` — `application/.../event/CompensationEvents.kt`
- `CommuteBookedCompensatedEvent` — `application/.../event/CompensationEvents.kt`
- `CommuteBookingCanceledCompensatedEvent` — `application/.../event/CompensationEvents.kt`
- `TravelOfferBookedCompensatedEvent` — `application/.../event/CompensationEvents.kt`
- `TravelOfferBookingCanceledCompensatedEvent` — `application/.../event/CompensationEvents.kt`

If you’d like this list exported as plain names only (one per line) or in another format (CSV/JSON), let me know.




### Events with explicit subscribers (via `EventBus.subscribe<T>()`)  
  
Below are all event classes that have at least one explicit subscriber registered for that exact type in the codebase, with the file where the subscription occurs.  
  
#### Application: `TravelOfferEventHandler.kt`  
File: `application/src/main/kotlin/pl/szymanski/wiktor/ta/eventHandler/TravelOfferEventHandler.kt`  
- `TravelOfferReservedEvent`  
- `TravelOfferReleaseEvent`  
- `CommuteExpiredEvent`  
- `AccommodationExpiredEvent`  
- `AttractionExpiredEvent`  
- `CommuteFullEvent`  
- `AccommodationBookedEvent`  
- `AttractionFullEvent`  
- `CommuteAvailableEvent`  
- `AccommodationBookingCanceledEvent`  
- `AttractionAvailableEvent`  
- `BookingCreatedEvent`  
- `BookingSagaStartedEvent`  
- `BookingSagaCompletedEvent` (two handlers subscribe to it)  
- `BookingSagaFailedEvent`  
- `BookingCancelSagaStartedEvent`  
- `BookingCancelSagaCompletedEvent` (two handlers subscribe to it)  
- `BookingCancelSagaFailedEvent`  
- `BookingCancelRequestedEvent`  
  
#### Application: `DateMetEventHandler.kt`  
File: `application/src/main/kotlin/pl/szymanski/wiktor/ta/eventHandler/DateMetEventHandler.kt`  
- `CommuteDateMetEvent`  
- `AccommodationDateMetEvent`  
- `AttractionDateMetEvent`  
  
#### Application: `OfferMaker.kt`  
File: `application/src/main/kotlin/pl/szymanski/wiktor/ta/offerMaker/OfferMaker.kt`  
- `TravelOfferExpiredEvent`  
  
### Notes  
- The above list is derived from searching for `EventBus.subscribe<...>` calls and enumerating their generic type arguments.  
- Commented-out subscriptions (e.g., the disabled handlers for `TravelOfferReserveFailedEvent` and `TravelOfferBookFailedEvent`) are not included because they are inactive.  
- If you want this list grouped by domain vs. application-level events, or exported as plain names, CSV, or JSON, I can format it accordingly.