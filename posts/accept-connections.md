# Accept connections

Server needs to accept clients' connections. First server creates an `acceptor`:  

	......
	boost::asio::io_context io_context;
        boost::asio::ip::tcp::acceptor acceptor{
            io_context,
            boost::asio::ip::tcp::endpoint{boost::asio::ip::tcp::v6(), 3303}};
	......
`boost::asio::ip::tcp::acceptor` is an instance of `basic_socket_acceptor`:  

	class tcp
	{
	......
	  /// The TCP acceptor type.
	  typedef basic_socket_acceptor<tcp> acceptor;
	......
	}

The following constructor of `basic_socket_acceptor` combines creating socket,  setting reuse address, binding & listening functions:  

	basic_socket_acceptor(boost::asio::io_context& io_context,
	    const endpoint_type& endpoint, bool reuse_addr = true)
	  : basic_io_object<BOOST_ASIO_SVC_T>(io_context)
	{
	......
	}

Then `acceptor` will accept the clients' connections. Following simple example just shows client's address and close the connection:  

	#include <boost/asio.hpp>
	#include <iostream>
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	        boost::asio::ip::tcp::acceptor acceptor{
	            io_context,
	            boost::asio::ip::tcp::endpoint{boost::asio::ip::tcp::v6(), 3303}};
	
	        while (1)
	        {
	            boost::asio::ip::tcp::socket socket{io_context};
	            acceptor.accept(socket);
	
	            std::cout << socket.remote_endpoint() << " connects to " << socket.local_endpoint() << '\n';
	        }
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << '\n';
	        return -1;
	    }
	
	    return 0;
	}

The running result is like this:  

	[::ffff:10.217.242.61]:39290 connects to [::ffff:192.168.35.145]:3303
	......
