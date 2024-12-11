# Secure Connection

- research: most secure way to...secure a TCP connection
	- **answer**: after some small research TLS 1.3 seems to be the way to go. Check the TLS OWASP cheatsheet in one of the links below.
	- links:
	    - https://security.stackexchange.com/questions/5126/whats-the-difference-between-ssl-tls-and-https
	    - https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html
	    - https://security.stackexchange.com/questions/241493/is-there-any-solution-beside-tls-for-data-in-transit-protection
	    - https://softwareengineering.stackexchange.com/questions/271366/tls-alternatives-that-do-not-require-a-central-authority
- research: what Zig supports to secure a TCP connection
	- current search: https://search.brave.com/search?q=%22io_uring%22+%22tls%22&source=web
	- links:
	    - https://ziglang.org/documentation/master/std/#std.crypto.tls
	    - https://github.com/ziglang/zig/issues/14171
	    - https://github.com/ziglang/zig/pull/19308
	    - https://github.com/mattnite/zig-mbedtls
			- https://github.com/Mbed-TLS/mbedtls
	    - https://harshityadav95.medium.com/tls-termination-proxy-3733b69680cb
	    - https://tls13.xargs.org/
	    - https://docs.kernel.org/networking/tls-offload.html
			- https://www.kernel.org/doc/html/latest/networking/tls.html
	    - https://github.com/fantix/kloop
		- https://github.com/crazyguitar/ktls-example (contains other links)
			- https://words.filippo.io/playing-with-kernel-tls-in-linux-4-13-and-go/
	- notes: https://words.filippo.io/playing-with-kernel-tls-in-linux-4-13-and-go/
		- claims about ktls:
			- only supports TLS 1.2
			- only supports encrypting packets, there's a number of things it defers to userspace applications: "Handshake, key exchange, certificate handling, alerts and
			renegotiation are left out of kernelspace"
			- only supports encryption, not decryption
			- "These limitations are very good to contain complexity and attack surface, but they mean that kTLS won't replace any userspace complexity as you still need a TLS
			library to do the handshake, for all other cipher suites, and for the receiving side of the connection. That makes kTLS purely a performance feature."
- research: how to implement a secure TCP connection in Zig
