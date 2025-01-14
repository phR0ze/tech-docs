# Nix Binary Cache <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

A binary cache receives requests for Nix packages and builds them if needed caching the result for 
other machines to use. Any machine with Nix installed can be a binary cache for another one, no 
matter the operating system Nix is running on.

### Quick links
* [Configure binary cache server](#configure-binary-cache-server)
* [Configure binary cache clients](#configure-binary-cache-clients)
* [Test the binary cache](#test-the-binary-cache)

## Configure binary cache server
`nix-serve` provides support for the binary cache protocol via HTTP. The binary cache will end up in 
your configuration as a substituter which then ends up in `/etc/nix/nix.conf`.

**References**
- [Binary Cache - NixOS Wiki](https://nixos.wiki/wiki/Binary_Cache)

1. Generate a `Ed25519` keypair for `nix-serve` to sign packages.
   ```bash
   mkdir -p include/var/lib/cache
   cd include/var/lib/cache
   # Note `cache` can be any unique value and is recommended to be the hostname of your cache
   nix-store --generate-binary-cache-key cache private.dec.pem public.pem
   sops encrypt --input-type binary --output private.enc.pem private.dec.pem
   sudo chown nix-serve private.dec.pem
   sudo chmod 600 private.dec.pem
   ```
2. Configure `nix-serve`
   ```nix
   services.nix-serve = {
     enable = true;
     secretKeyFile = "/var/lib/cache/private.dec.pem";
   };
   ```
3. Configure `nginx` to serve the virtual hostname and allow port `80` through the firewall
   ```nix
   services.nginx = {
     enable = true;
     recommendedProxySettings = true;
     virtualHosts."cache" = {
       locations."/".proxyPass = "http://${config.services.nix-serve.bindAddress}:${toString config.services.nix-serve.port}";
     };
   };

   networking.firewall.allowedTCPPorts = [
     config.services.nginx.defaultHTTPListenPort
   ];
   ```
4. Test
   ```bash
   $ curl http://[IP-ADDRESS]/nix-cache-info
   # and
   $ nix path-info -r /nix/store/sb7nbfcc1ca6j0d0v18f7qzwlsyvi8fz-ocaml-4.10.0 --store https://[IP-ADDRESS]
   ```

## Configure binary cache clients
Add the new binary cache as a `substituters` and `extra-substituters` and add the public key to the 
`trusted-public-keys`.

```nix
nix = {
  settings = {
    substituters = [
      "http://binarycache.example.com"
      "https://nix-community.cachix.org"
      "https://cache.nixos.org/"
    ];
    trusted-public-keys = [
      "binarycache.example.com-1:dsafdafDFW123fdasfa123124FADSAD"
      "nix-community.cachix.org-1:mB9FSh9qf2dCimDSUo8Zy7bkq5CX+/rkCWyvRCYg3Fs="
    ];
  };
};
```

## Test the binary cache
1. Check the cache is up and accessible
   ```bash
   $ curl http://$IP_ADDRESS/nix-cache-inf
   ```
2. Check that its built on the cache server
   ```bash
   $ hash=$(nix-build '<nixpkgs>' -A pkgs.hello | awk -F '/' '{print $4}' | awk -F '-' '{print $1}')
   this path will be fetched (0.00 MiB download, 0.22 MiB unpacked):
     /nix/store/1q8w6gl1ll0mwfkqc3c2yx005s6wwfrl-hello-2.12.1
   copying path '/nix/store/1q8w6gl1ll0mwfkqc3c2yx005s6wwfrl-hello-2.12.1' from 'http://192.168.1.50'...
   ```
3. Check with th ehash we constructed to see if the signature is being included
   ```bash
   $ curl "http://$IP_ADDRESS/$hash.narinfo" | grep "Sig: "
   ```
