## Contrato inteligente: Semáforo

## Inicio: Clonar el template para contratos inteligentes

**comando:**
```bash
git clone https://github.com/Vara-Lab/AsynchronousContractTemplate.git
```

## Directorio IO

### PASO 1 Definir las acciones para el semáforo.
```rust
#[derive(Decode, Encode, TypeInfo)]
#[codec(crate = gstd::codec)]
#[scale_info(crate = gstd::scale_info)]
pub enum TrafficLightAction {
    Green,
    Yellow,
    Red
}

```

### PASO 2 Definir las eventos para el semáforo.
```rust
#[derive(Decode, Encode, TypeInfo)]
#[codec(crate = gstd::codec)]
#[scale_info(crate = gstd::scale_info)]
pub enum TrafficLightEvent {
    Green,
    Yellow,
    Red
}

```

### PASO 3 Definir las acciones, estado y eventos como Metadata.
```rust

#[derive(Decode, Encode, TypeInfo)]
#[codec(crate = gstd::codec)]
#[scale_info(crate = gstd::scale_info)]
pub struct IoTrafficLightState {
    pub current_light: String,
    pub all_users: Vec<(ActorId, String)>,
}

pub struct ContractMetadata;

impl Metadata for ContractMetadata {
    type Init = ();
    type Handle = InOut<TrafficLightAction, TrafficLightEvent>;
    type Others = ();
    type Reply = ();
    type Signal = ();
    type State = Out<IoTrafficLightState>;
}

```


## Directorio src

### PASO 1 Definir el estado principal del semáforo dentro de la variable estática mutable STATE.
```rust

static mut STATE:Option<TrafficLightState> = None;

#[derive(Clone, Default)]
pub struct TrafficLightState {
    pub current_light: String,
    pub all_users: HashMap<ActorId, String>,
}

```

### PASO 2 Definir definir implementaciones de ser necesarias.
```rust
impl TrafficLightState {
    #[allow(dead_code)]
    async fn firstmethod(&mut self) {}
    #[allow(dead_code)]
    async fn secondmethod(&mut self) {}
    #[allow(dead_code)]
    async fn thirdmethod(&mut self) {}
}
```

### PASO 3 Definir en el interior de la función init() para inicializar el estado principal.
```rust
#[no_mangle]
extern "C" fn init() {
    
    let state = TrafficLightState {
        ..Default::default()
    };

    unsafe { STATE = Some(state) };
}

```

### PASO 4 Definir en el interior de la función Handle y definimos Acción->Transición->Evento.
**comando:**
```rust
#[async_main]
async fn main() {
    
    // We load the input message
    let action: TrafficLightAction = msg::load().expect("Could not load Action");

    let state = unsafe { STATE.as_mut().expect("Unexpected error in factory_state") };

    // We receive an action from the user and update the state. Example:
    match action {
        TrafficLightAction::Green => {


            state.current_light = "Green".to_string();

            // Update your second field in state.
            state.all_users.insert(msg::source(), "Green".to_string());


             // Generate your event.
             let _ =msg::reply(TrafficLightEvent::Green,0);


        }
        TrafficLightAction::Yellow => {


            // Update your first field in state.       
            state.current_light = "Yellow".to_string();

            // Update your second field in state.
            state.all_users.insert(msg::source(), "Yellow".to_string());


             // Generate your event.
             let _ =msg::reply(TrafficLightEvent::Yellow,0);

           

        }
        TrafficLightAction::Red => {
           
            
           // Update your first field in state.       
           state.current_light = "Red".to_string();

           // Update your second field in state.
           state.all_users.insert(msg::source(), "Red".to_string());


            // Generate your event.
            let _ =msg::reply(TrafficLightEvent::Red,0);
        }

    };
}

```

## Directorio State

### PASO 1 Definir las implementaciones para el estado.
```rust
#![no_std]

use io::*;
use gmeta::{ Metadata, metawasm};
use gstd::{ ActorId, prelude::*};

#[cfg(feature = "binary-vendor")]
include!(concat!(env!("OUT_DIR"), "/wasm_binary.rs"));

#[metawasm]
pub mod metafns {
    pub type State = <ContractMetadata as Metadata>::State;


    // Add your State functions

    pub fn state(state: State) -> IoTrafficLightState{
        state
    }

}

```

## Despliega el contrato en la plataforma IDEA e interactua con tu contrato.