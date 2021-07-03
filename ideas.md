# High level ideas / thoughts / notes

This is a bit messy, it will be split out later to seperate docs, just writing down initial ideas

- Based on the latest longterm stable Linux Kernel (as of July 2021, 5.10.47)

- It will not be intended for small embedded systems (think typical OpenWRT devices)
  - It's fine if it works, but should not limit design choices and small embedded devices are not in the scope of this project
  - Focus is on being installed on native x86 64bit hardware with >4Gb memory, or running it as a VM
    - ARM64 might work, but not a focus initially
    - Mostly down to which architectures the Linux Kernel / Dotnet / Rust supports.
  - That is not to say resources should be wasted, but the focus should be on speed (network/appliance services), not on minimizing resource usage (cpu/mem)
    
- Supporting a "Central" appliance, that can manage multiple sites
  - Supporting templates from the "Central" appliance
    - Firewall rules, services, etc
    - Optionally sending logs upstream, to the central appliance
  - But should function equally well as a standalone appliance
  
- The focus is on core network services
  - In scope
    - Firewall
    - Routing (IPv4/IPv6)
    - VPN (Wireguard, OpenVPN)
    - Support services like DHCP/Radius/Auth
    - IDS/IPS (not initially, but in scope)
    - Load balancing (not initially, but somewhat in scope)    
  - Out of scope
    - Not trying to be a swizz army knife doing a little bit of everything
    - Installing 3rd party packages (even if initially possible due to not being a dedicated appliance in the start)
    - Running containers
    - Running "download" clients

- Implemented with an optional web interface, that calls an backend API, all functionality should be provided by the API
  - Makes it easy to see what API calls are used and how, for any action done on the Web interfaces
  - Also enables changing settings from infrastructure as code, without requiring using the web interface itself
  - This applies to both the "Central" appliance, and each indivual standalone installs
  - Allows adding a local CLI as well

- Webinterface/API built using C# with .NET 5.0 (6.0 when released), using ASP.NET Core MVC
  - Might be replaced by Rust as well, but havent found any real good comparable frameworks yet, so TBD in the future, but initally will be .NET based.
    - .NET limits the CPU architectures supported (32/64bit x86, ARM64).
    - Most of the actual work is done from the backend daemons, so not the end of the world to replace later on.

- Backend daemons built using Rust
  - Speed, and safety features
  - Widely available on different CPU architectures
  - Requires no special runtime installed
  - Using gRPC for RPC between the API and the backend daemons
  - Responsible for changing OS settings, firewall rules, etc
  - Responsible for reporting live metrics to the Webinterface (and possible to the API as well TBD)
  - Calling native functions, so not just calling out to shell commands
    - So fx calling netlink directly, rather than relying on "ip <cmds>" etc
    
- Making sure to prioritise security
  - Follow best practices, with least amount of privileges required
    - Backend services requires more privileges to function, Web/API/(CLI) shouldn't
  - Minimum TLS 1.2

- Using YAML for configuration/settings
  - Easier to read for humans
  - TBD for the actual "appliance" config
    - But should be diffable between changes to settings/firewall rules, between diffrent config commits.
    - Changes should be stored in Git

# How to run the standalone/central appliance

## During Development
- Installable on a minimum Ubuntu LTS/Debian testing

## Production
- Create our own minimal Distro, based on apt, probably ubuntu/debian packages
