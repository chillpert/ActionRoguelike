# Notes

## Debugging

- Use "DrawDebugDirectionalArrow" instead of "DrawDebugLine"
- It seems like at least on Windows with VS and `Development Game Editor` profile enabled, breakpoints are still working, so basically not only in `Debug Game Editor` 
  - But it cannot display many symbols, so its usefulness is restricted
  - Although, it seems like symbols of the game itself are working

## Engine

- Control rotation is separate from pawn rotation, e.g. the camera in FPS might move up and down but the character should only move its head
- There is `AActor::TeleportTo` as an alternative to `AActor::SetActorLocation`, which will do some extra queries to check if the actor can be moved there

### Delegates

- Reminder not to forget to add `UFUNCTION` to functions you are binding to a delegate, because they need to be exposed to the UE property system
- You can actually bind delegates in the constructor too
- Binding delegates of an actor's owned components is best done in `AActor::PostInitializeComponent`, but `AActor::BeginPlay` also is valid

### String types

- `FString`
  - Mostly used for debugging
  - Allows string manipulation (append, split, ...)
- `FName`
  - Hashed for faster 'string' comparison
  - Used by system and gameplay logic
  - Doesn't change once assigned
- `FText`
  - Front-end text to display in UI
  - Easily 'localized' into different languages

- ⚠️ Start using `FName` everywhere it is possible

### Logging

- The `TEXT(x)` macro allows for a larger character set (Unicode, instead of ANSI) and converts the literal to the type that UE expects, and is generally useful when displaying text in a different language
- `UWorld::TimeSeconds` can be used to retrieve the time passed since the game starts, as an alternative to using functions from `UKismetSystemLibrary`
- Use `DrawDebugString` to display strings in 3D (in-game) instead of using an UI element
- Use `FString::Printf` to create a string which can include other types, similar to `UE_LOG(LogTemp, Log, TEXT("My float: %f"), MyFloat);`
- Use `UE_LOGFMT` instead of `UE_LOG`
  - It does not require `TEXT()`
  - It has named and unnamed options, e.g. `UE_LOGFMT(LogTemp, Log, "My array: {array}", ("array", MyArray));` or `UE_LOGFMT(LogTemp, Log, "My array: {0}", MyArray);`

### Collision

- For collisions between two channels, always the weakest is used (ignore < overlap < block)
  - Suppose object A (`WorldStatic` | Ignore channel `Projectile`) and object B (`Projectile` | Block channel `WorldStatic`), then both will ignore each other
- Consider using AActor::GetActorEyesViewPoint(FVector&, FRotator&) as a starting point for tracing (this requires APawn::BaseEyeHeight to be set)
- Consider replacing line traces with sweeps to make it less likely for traces to fail, e.g. in the context of player-world interactions
  - `UWorld::SweepMultiByObjectType(...)`

### Interfaces

- Use `UMyInterface` when checking if actor implements it ...
  - `MyActor->Implements<UMyInterface>`
- ... but use `IMyInterface` when executing functions
  - `IMyInterface::Execute_MyFunction(Params ...)`

### Animations

#### Timelines

- Consider using a timeline's event track (or other tracks than float track for that matter)
- Keep in mind that `Ignore Time Dilation` exists, in situations where timelines are buggy

### AI

- `UPawnSensingComponent` is simpler but older version of `AIPerceptionComponent`
- The number in braces next to a failed EQS query item is actually the index of the test that failed
- You should use the visual debugger more often, as it also saves the EQS results
    - It also does not need to be enabled via code, just hit record

## Editor 

### UMG

- Widgets we already created, can be used in other widgets
- In the event graph, get the player with `Get Owning Player Pawn` and the controller with `Get Owning Player`

### Blueprints

- Do not forget about right-clicking variables in the event graph, and select `Watch variable` (you must have the BP visible while playing to see)
- Do not forget about gates and multi gates
- Consider using `BlueprintReadOnly` more often

### Shortcuts

- Use `Shift + F1` to get mouse back
  - This is not the same as `F8`, since `F8` will eject

### Customizations

- It is possible to change the loading screen, when launching your project (see ActionRogueLike)

### Materials

- Use `DebugScalarValues` function to display the actual numbers that nodes in the material graph output
  - Simply plug the result of that node into the base color
- You can also directly set parameters of a material in BPs with the a mesh component as target, e.g. `Set Scalar Parameter Value on Materials`
  - It even works without a dynamic material instance or a material instance asset to begin with
- You should try to avoid setting material parameters via Blueprint, because it requires the data to be uploaded to the GPU which is expensive
  - If the material can be driven with only built-in nodes, it will be faster
- Using material instances, we can tweak values while the game is running and it will update in real-time
