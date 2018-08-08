# Install Boost

Installing [Boost](https://www.boost.org/) is not hard. On `OpenBSD`:  

	$ pkg_add boost

While on `Arch Linux`:  

	$ sudo pacman -S boost

The thing is when compiling program, please link related `Boost` libraries. E.g., on `OpenBSD`:  

	$ c++ -L/usr/local/lib client.cpp -o client -lboost_system

On `Arch Linux`, `-pthread` option is needed:  

	$ c++ -pthread client.cpp -o client -lboost_system