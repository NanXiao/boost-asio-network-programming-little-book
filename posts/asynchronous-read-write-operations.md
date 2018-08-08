# Asynchronous read/write operations

Unlike classical `UNIX` socket programming, `boost.asio` has battery-included asynchronous read/write abilities. Still use `basic_stream_socket` as an example, and one pair of the implementations is like this:   

	template <typename ConstBufferSequence, typename WriteHandler>
	  BOOST_ASIO_INITFN_RESULT_TYPE(WriteHandler,
	      void (boost::system::error_code, std::size_t))
	  async_send(const ConstBufferSequence& buffers,
	      BOOST_ASIO_MOVE_ARG(WriteHandler) handler)
	{
		.......
	}
	template <typename MutableBufferSequence, typename ReadHandler>
	  BOOST_ASIO_INITFN_RESULT_TYPE(ReadHandler,
	      void (boost::system::error_code, std::size_t))
	  async_receive(const MutableBufferSequence& buffers,
	      BOOST_ASIO_MOVE_ARG(ReadHandler) handler)
	{
		.......
	}

Since `async_send` and `async_receive` functions will return immediately, and not block current thread, you should pass a callback function as the parameter which receives the result of read/write operations:  

	void handler(
		const boost::system::error_code& error, // Result of operation.
		std::size_t bytes_transferred           // Number of bytes processed.
	)

There is a simple client/server example. Below is client code:  

	#include <boost/asio.hpp>
	#include <functional>
	#include <iostream>
	#include <memory>
	
	void callback(
	        const boost::system::error_code& error,
	        std::size_t bytes_transferred,
	        std::shared_ptr<boost::asio::ip::tcp::socket> socket,
	        std::string str)
	{
	    if (error)
	    {
	        std::cout << error.message() << '\n';
	    }
	    else if (bytes_transferred == str.length())
	    {
	        std::cout << "Message is sent successfully!" << '\n';
	    }
	    else
	    {
	        socket->async_send(
	                boost::asio::buffer(str.c_str() + bytes_transferred, str.length() - bytes_transferred),
	                std::bind(callback, std::placeholders::_1, std::placeholders::_2, socket, str));
	    }
	}
	
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	
	        boost::asio::ip::tcp::endpoint endpoint{
	                boost::asio::ip::make_address("192.168.35.145"),
	                3303};
	
	        std::shared_ptr<boost::asio::ip::tcp::socket> socket{new boost::asio::ip::tcp::socket{io_context}};
	        socket->connect(endpoint);
	
	        std::cout << "Connect to " << endpoint << " successfully!\n";
	
	        std::string str{"Hello world!"};
	        socket->async_send(
	                boost::asio::buffer(str),
	                std::bind(callback, std::placeholders::_1, std::placeholders::_2, socket, str));
	        socket->get_executor().context().run();
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << '\n';
	        return -1;
	    }
	
	    return 0;
	}

Let's go through the code:  

(1) Since socket object is non-copyable (please refer [socket](socket.md)), socket is created as an shared pointer:  

	......
	std::shared_ptr<boost::asio::ip::tcp::socket> socket{new boost::asio::ip::tcp::socket{io_context}};
	......

(2) Because the callback only has two parameters, it needs to use `std::bind` to pass additional parameters:  

	......
	std::bind(callback, std::placeholders::_1, std::placeholders::_2, socket, str)
	......

(3) `async_send` does not guarantee all the bytes are sent (`boost::asio::async_write` returns either all bytes are sent successfully or an error occurs), so needs to reissue `async_send` in callback:  

	......
	if (error)
	{
	    ......
	}
	else if (bytes_transferred == str.length())
	{
	    ......
	}
	else
	{
	    socket->async_send(......);
	}
(4) `io_context.run` function will block until all work has finished and there are no
more handlers to be dispatched, or until the `io_context` has been stopped:  

	socket->get_executor().context().run();
If there is no `io_context.run` function, the program will exit immediately.  

Check the server code who uses `async_receive`:  

	#include <ctime>
	#include <functional>
	#include <iostream>
	#include <string>
	#include <boost/asio.hpp>
	
	void callback(
	        const boost::system::error_code& error,
	        std::size_t,
	        char recv_str[]) {
	    if (error)
	    {
	        std::cout << error.message() << '\n';
	    }
	    else
	    {
	        std::cout << recv_str << '\n';
	    }
	}
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	
	        boost::asio::ip::tcp::acceptor acceptor(
	                                        io_context,
	                                        boost::asio::ip::tcp::endpoint(boost::asio::ip::tcp::v4(), 3303));
	
	        for (;;)
	        {
	            boost::asio::ip::tcp::socket socket(io_context);
	            acceptor.accept(socket);
	
	            char recv_str[1024] = {};
	            socket.async_receive(
	                    boost::asio::buffer(recv_str),
	                    std::bind(callback, std::placeholders::_1, std::placeholders::_2, recv_str));
	            socket.get_executor().context().run();
	            socket.get_executor().context().restart();
	        }
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << std::endl;
	    }
	
	    return 0;
	}
	
There are two caveats you need to pay attention to:  

(1) Just for demo purpose: for every client, the callback is called only once;  
(2) `io_context.restart` must be called to invoke another `io_context.run`.  

Correspondingly, you can also check how to use `boost::asio::async_read`.

Build and run programs. Client outputs following:  

	$ ./client
	Connect to 192.168.35.145:3303 successfully!
	Message is sent successfully!

Server outputs following:  

	$ ./server
	Hello world!
