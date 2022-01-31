# Build your transaction, sign, and send it to the Algorand network in Rust!

Here in this article I will teach you how to make, sign, and broadcase a transaction to the Algorand network in Rust!


### Preparations

#### Rust
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

#### create the project with Cargo
```
cargo new Algorand_Transaction_Sender
```

#### Dependency
Add the following dependency to Cargo.toml like this 
```
[dependencies]
algonaut = {git = "https://github.com/manuelmauro/algonaut", rev = "11808fa3650712bbd903560306400730ecc1d7b5"}
```

#### Replace the main.rs with my code:


```
use algonaut::core::{Address, MicroAlgos};
use algonaut::transaction::{BaseTransaction, Payment, Transaction, TransactionType};
use algonaut::{Algod, Kmd};
use std::error::Error;


fn main() -> Result<(), Box<dyn Error>> {
    let kmd = Kmd::new() // we use the kmd algorand daemon
        .bind("http://localhost:4002")
        .auth("aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa")
        .client_v1()?;


        // now we are locating our default wallet and obtain a handle to it.

        let list_response = kmd.list_wallets()?; // 

        let wallet_id = match list_response
            .wallets
            .into_iter()
            .find(|wallet| wallet.name == "unencrypted-default-wallet")
        {
            Some(wallet) => wallet.id,
            None => return Err("Wallet not found".into()),
        };
        println!("Wallet: {}", wallet_id); //  In this example, our wallet has no password, so we just use a blank string
    
        let init_response = kmd.init_wallet_handle(&wallet_id, "")?;
        let wallet_handle_token = init_response.wallet_handle_token;


        //let's build a payment transaction

        let algod = Algod::new() // we use algod client or identifying our private network
        .bind("http://localhost:4001")
        .auth("aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa")
        .client_v1()?;

        let transaction_params = algod.transaction_params()?;
        let genesis_id = transaction_params.genesis_id;
        let genesis_hash = transaction_params.genesis_hash;


       // let's specify who is the sender and the receiver
        let public_key = "INSERT-HERE-YOUR-ADDRESS";
        let to_address = Address::from_string(public_key.as_ref())?;
        let from_address = Address::from_string(public_key.as_ref())?;
        //We used the same address for simplicity, but you can substitute them
        println!("Receiver: {:#?}", to_address);


        let base = BaseTransaction {
            sender: from_address, // you must have a positive balance on this wallet, run: ./sandbox goal account list
            first_valid: transaction_params.last_round,
            last_valid: transaction_params.last_round + 1000,
            note: Vec::new(),
            genesis_id,
            genesis_hash,
        };
        println!("Base: {:#?}", base);


      // now is time to build our base transaction struct  
        let payment = Payment {
            amount: MicroAlgos(100_000),
            receiver: to_address,
            close_remainder_to: None,
        };
        println!("Payment: {:#?}", payment);


        let transaction =
        Transaction::new_flat_fee(base, MicroAlgos(1_000), TransactionType::Payment(payment));
    println!("Transaction: {:#?}", transaction);

    // now we have the two components of our transaction, so now it's ready to fire!

    //  Before broadcasting the transaction to the network, we must sign it, how?

    let sign_response = kmd.sign_transaction(&wallet_handle_token, "", &transaction)?;
    println!("Signed: {:#?}", sign_response); // using the KDM sdk for signing, passing the token wallet, 
    // the transaction, and its pass 

    let send_response = algod.raw_transaction(&sign_response.signed_transaction)?; // using algod for broadcasting
    // there we go!, its already broadcasted!
    println!("Transaction ID: {}", send_response.tx_id);


    Ok(())
}
```

#### Run the code
```
cargo run
```
