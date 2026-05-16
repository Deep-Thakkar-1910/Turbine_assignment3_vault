# Vault Program on Solana

A Solana program built with Anchor that allows users to create personal SOL vaults secured by Program Derived Addresses (PDAs). Each vault is owned by a single authority who can deposit, withdraw, and close the vault. Anyone can deposit into an existing vault, but only the authority can withdraw or close it.

---

## What's Inside

### Program Instructions

| Instruction        | What it does                                                                                      |
| ------------------ | ------------------------------------------------------------------------------------------------- |
| `initialize`       | Creates a new vault state PDA and vault PDA for the signing authority                             |
| `deposit(amount)`  | Transfers `amount` lamports from the signer into the vault via CPI                                |
| `withdraw(amount)` | Transfers `amount` lamports from the vault back to the authority (authority-gated, rent-safe)     |
| `close`            | Drains all remaining lamports from the vault and closes the vault state account (authority-gated) |

### Architecture

```
Authority (signer)
    │
    ├── VaultState PDA  [seeds: "vault" + authority]
    │       ├── authority: Pubkey
    │       ├── state_bump: u8
    │       └── vault_bump: u8
    │
    └── Vault PDA       [seeds: "vault" + vault_state]
            └── (holds SOL lamports)
```

### Error Codes

| Error                 | When it triggers                                                         |
| --------------------- | ------------------------------------------------------------------------ |
| `InsufficientBalance` | Withdrawal amount exceeds available balance after rent-exemption reserve |
| `InvalidAuthority`    | Signer is not the vault's designated authority                           |

---

## Tech Stack

- **Language** - Rust (toolchain `1.89.0`)
- **Framework** - [Anchor](https://www.anchor-lang.com/) v1.0.0
- **Testing** - [LiteSVM](https://github.com/LiteSVM/litesvm) (fast local Solana VM, no validator needed)
- **Network** - Localnet (configurable in `Anchor.toml`)

---

## Prerequisites

1. **Rust** installed via [rustup](https://rustup.rs/) (toolchain 1.89.0 is pinned in `rust-toolchain.toml`)
2. **Solana CLI** installed (v2.x):
   ```bash
   sh -c "$(curl -sSfL https://release.anza.xyz/stable/install)"
   ```
3. **Anchor CLI** installed (v1.0.0+):
   ```bash
   cargo install --git https://github.com/coral-xyz/anchor avm --force
   avm install latest
   avm use latest
   ```
4. **Node.js** installed (for Anchor workspace tooling)
5. A Solana keypair at the default path:
   ```
   ~/.config/solana/id.json
   ```

---

## Getting Started

```bash
# Clone the repo
git clone https://github.com/Deep-Thakkar-1910/Turbine_assignment3_vault.git
cd vault_program

# Install JS dependencies
npm install

# Build the program
anchor build

# Run tests
anchor test
```

---

## Tests

Tests are written in Rust using LiteSVM for fast local execution without spinning up a validator.

| Test                                            | What it verifies                                                                      |
| ----------------------------------------------- | ------------------------------------------------------------------------------------- |
| `test_initialize_deposit_withdraw_close`        | Full lifecycle: init vault, deposit 5 SOL, withdraw 4 SOL, close vault, verify refund |
| `test_withdraw_more_than_balance`               | Expects `InsufficientBalance` when withdrawing more than deposited                    |
| `test_withdraw_from_different_authority`        | Expects `InvalidAuthority` when a non-owner attempts withdrawal                       |
| `test_close_from_different_authority`           | Expects `InvalidAuthority` when a non-owner attempts to close the vault               |
| `test_deposit_from_different_authority_allowed` | Verifies that anyone can deposit SOL into an existing vault                           |

### Running Tests

```bash
anchor test
```

---

## Project Structure

```
vault_program/
├── programs/vault_program/
│   ├── src/
│   │   ├── lib.rs              # Program entrypoint and instruction dispatch
│   │   ├── state.rs            # VaultState account definition
│   │   ├── constants.rs        # Seeds and size constants
│   │   ├── error.rs            # Custom error codes
│   │   └── instructions/
│   │       ├── mod.rs
│   │       ├── initialize.rs   # Vault creation logic
│   │       ├── deposit.rs      # SOL deposit via CPI transfer
│   │       ├── withdraw.rs     # Authority-gated withdrawal with rent check
│   │       └── close.rs        # Drain vault and close state account
│   └── tests/
│       └── test_initialize.rs  # LiteSVM integration tests
├── Anchor.toml
├── Cargo.toml
├── package.json
└── README.md
```

---

## Program ID

```
FV451y9S6Pe6qNFbBrdyJkdAY6mcsEex6Mjf49aKjuY2
```

---
