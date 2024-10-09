---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
mermaid: true
---

# Flag forge

A site dedicated to learning about common CTF exploits and tools.


```mermaid
mindmap
  root((Flag Forge))
    Crypto
      Symmetrical
        AES
      Public Key
        RSA
        Diffie-Hellman
        Elliptical Curves
      Hash
    Forensics
      Logs
      Disk Image
      Network
        pcap file
      Memory dump
    Misc
      Recon
      Guessing
    Pwn
      Stack based buffer overflow
      Heap based buffer overflow
      Executable Stack
      ROP Chain
      Format string
    Reverse
      Languages
        C
        C++
        Rust
        Go
        Esolangs
      Obfuscation
        Packing
        Virtualization

      Anti-Debug
        Ptraceme
        Checksum
        Nanomites

      Concepts
        Calling conventions
        Structures
        Cryptography primitives
        Games

    Web
      Client
        XSS
      Server
        Injections
          SQLi
          Template injection
```

<script>
      document.addEventListener('click', function (event) {
        if( event.target.tagName === "tspan") {
            let mindmapdiv = document.getElementsByClassName("language-mermaid")[0];
            if (!mindmapdiv.contains(event.srcElement) ){
                return;
            }
            let elem = event.srcElement;
            while (elem && mindmapdiv.contains(elem)) {
                if (elem.classList.contains("mindmap-node")) {
                    event.preventDefault();
                    alert(elem.textContent);
                    return;
                }
                elem = elem.parentElement;
            }
        }})
</script>
