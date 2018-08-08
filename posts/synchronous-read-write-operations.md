# Synchronous read/write operations

Once the connection is established, the client and server can communicate with each other. Like classical `UNIX` socket programming, `boost::asio` also provides `send` and `receive` functions. Use `basic_stream_socket` as an example and one pair of the implementations is like this:     

	template <typename ConstBufferSequence>
  	std::size_t send(const ConstBufferSequence& buffers)
	{
		......
	}
	......
	template <typename MutableBufferSequence>
  	std::size_t receive(const MutableBufferSequence& buffers)
	{
		......
	}

Please notice the buffer types of `send/receive` are `ConstBufferSequence/MutableBufferSequence`, and we can use `boost::asio::buffer` function to construct related types.  

Below is a simple client program which sends "`Hello world!`" to server after connection is established:  

	#include <boost/asio.hpp>
	#include <iostream>
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	
	        boost::asio::ip::tcp::endpoint endpoint{
	                boost::asio::ip::make_address("10.217.242.61"),
	                3303};
	        boost::asio::ip::tcp::tcp::socket socket{io_context};
	        socket.connect(endpoint);
	
	        std::cout << "Connect to " << endpoint << " successfully!\n";
	
	        socket.send(boost::asio::buffer("Hello world!"));
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << '\n';
	        return -1;
	    }
	
	    return 0;
	}

There is the server program which waits receiving greeting from client:  

	#include <boost/asio.hpp>
	#include <iostream>
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	        boost::asio::ip::tcp::acceptor acceptor{
	                io_context,
	                boost::asio::ip::tcp::endpoint{boost::asio::ip::tcp::v4(), 3303}};
	
	        while (1)
	        {
	            boost::asio::ip::tcp::socket socket{io_context};
	            acceptor.accept(socket);
	
	            std::cout << socket.remote_endpoint() << " connects to " << socket.local_endpoint() << '\n';
	
	            char recv_str[1024] = {};
	            socket.receive(boost::asio::buffer(recv_str));
	
	            std::cout << "Receive string: " << recv_str << '\n';
	        }
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << '\n';
	        return -1;
	    }
	
	    return 0;
	} 

Build and run programs. Client outputs following:  

	$ ./client
	Connect to 10.217.242.61:3303 successfully!

Server outputs following:  

	$ ./server
	10.217.242.21:64776 connects to 10.217.242.61:3303
	Receive string: Hello world!

If no error occurs, `send` can guarantee at least one byte is sent successfully, and you should check the return value to see whether all bytes are sent successfully or not. `receive` is similar as `send`. `boost::asio::basic_stream_socket` also provides `read_some` and `write_some` which have the same functions as `receive` and `send`.  

If we don't bother to check the middle state (partial bytes are sent successfully), and only care whether all bytes are sent successfully or not, we can use `boost::asio::write` which actually uses `wrtie_some` under the hood.  Correspondingly, it is not hard to guess what `boost::asio::read`does.
