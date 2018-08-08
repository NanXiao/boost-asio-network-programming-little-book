# Endpoint

Endpoint is composed of "`IP` address + port":  

	basic_endpoint(const boost::asio::ip::address& addr, unsigned short port_num)
	    : impl_(addr, port_num)
	{
	}

Client uses endpoint to designate server address, and server application uses endpoint to identify which address will be used to listen and accept connections. An example of creating `TCP` endpoint is like this:  

	boost::asio::ip::tcp::endpoint endpoint{
            boost::asio::ip::make_address("127.0.0.1"),
            3303};

Usually, the server needs to listen to all the address of current machine, and it can resort to another constructor of `basic_endpoint`:  

	basic_endpoint(const InternetProtocol& internet_protocol,
	      unsigned short port_num)
	    : impl_(internet_protocol.family(), port_num)
	{
	}

An example of creating and `UDP` server who listens to all `IPv6` addresses:  

	boost::asio::ip::udp::endpoint endpoint{
            boost::asio::ip::udp::v6(),
            3303};