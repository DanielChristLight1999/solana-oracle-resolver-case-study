# Security & Public Disclosure Boundary

This repository documents proprietary blockchain automation at an architectural level.

## Never publish

The following categories remain private:

- Oracle signing keys
- Seed phrases
- RPC credentials
- API tokens
- Sensitive wallet addresses
- Production infrastructure details
- Private database contents
- Proprietary program source
- Client-confidential configuration

## Security model demonstrated by the case study

The resolver design emphasizes:

- Explicit oracle authorization
- State validation before state-changing actions
- Idempotent processing
- Bounded retries
- Separation of observation from decision logic

## Public portfolio boundary

The purpose of this repository is to demonstrate engineering judgment, backend architecture, and blockchain reliability patterns without reproducing the private production implementation.
