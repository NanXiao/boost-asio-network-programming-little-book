# UDP communication

We have discussed how to communicate through `TCP` enough, so it is time to switch to `UDP` now. `UDP` is a connectionless protocol, and it is easier to use than `TCP`. There is a client/server example. Below is client code:  

	#include <boost/asio.hpp>
	#include <iostream>
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	
	        boost::asio::ip::udp::socket socket{io_context};
	        socket.open(boost::asio::ip::udp::v4());
	
	        socket.send_to(
	                boost::asio::buffer("Hello world!"),
	                boost::asio::ip::udp::endpoint{boost::asio::ip::make_address("192.168.35.145"), 3303});
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << '\n';
	        return -1;
	    }
	
	    return 0;
	}

Although there is no need to call `socket.connect` function, you need call `socket.open` explicitly. Furthermore, the server's endpoint needs to be specified when invoking `socket.send_to`.  

Server code is like this:  

	#include <ctime>
	#include <functional>
	#include <iostream>
	#include <string>
	#include <boost/asio.hpp>
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	
	        for (;;)
	        {
	            boost::asio::ip::udp::socket socket(
	                    io_context,
	                    boost::asio::ip::udp::endpoint{boost::asio::ip::udp::v4(), 3303});
	
	            boost::asio::ip::udp::endpoint client;
	            char recv_str[1024] = {};
	
	            socket.receive_from(
	                    boost::asio::buffer(recv_str),
	                    client);
	            std::cout << client << ": " << recv_str << '\n';
	        }
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << std::endl;
	    }
	
	    return 0;
	}

Very easy, isn't it? Build and run client and server. The following log will be printed on server side:  

	$ ./server
	10.217.242.21:63838: Hello world!
	10.217.242.21:61259: Hello world!
