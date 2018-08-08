# Socket

There are `4` types of socket:

(1) `basic_stream_socket`:  
This socket provides sequenced, reliable, two-way connection based byte streams. `tcp::socket` is an instance of this socket:  

	class tcp
	{
	......
	  /// The TCP socket type.
	  typedef basic_stream_socket<tcp> socket;
	......
	}

(2) `basic_datagram_socket`:  
This socket provides connectionless, datagram service. `udp::socket` is an instance of this socket:  

	class udp
	{
	......
	  /// The UDP socket type.
  	  typedef basic_datagram_socket<udp> socket;
	......
	}
(3) `basic_raw_socket`:  
This socket provides access to internal network protocols and interfaces. `icmp::socket` is an instance of this socket:  

	class icmp
	{
	......
	  /// The ICMP socket type.
  	  typedef basic_raw_socket<icmp> socket;
	......
	}
(4) `basic_seq_packet_socket`:  
This socket combines stream and datagram: it provides a sequenced, reliable, two-way connection based datagrams service. [SCTP](https://en.wikipedia.org/wiki/Stream_Control_Transmission_Protocol) is an example of this type of service.  

All these `4` sockets derive from `basic_socket` class, and need to associate with an `io_context` during initialization. Take `tcp::socket` as an example:  

	boost::asio::io_context io_context;
	boost::asio::ip::tcp::socket socket{io_context};

Please notice the `io_context` should be a reference in constructor of `socket` (Please refer [io_context](io_context.md)). Still use  `basic_socket` an an instance, one of its constructor is like following:

	  explicit basic_socket(boost::asio::io_context& io_context)
	    : basic_io_object<BOOST_ASIO_SVC_T>(io_context)
	  {
	  }

For `basic_io_object` class, it does not support copy constructed/copy assignment:  

	......
	private:
	  basic_io_object(const basic_io_object&);
	  void operator=(const basic_io_object&);
	......

whilst it can be movable:  
	
	......
	protectd:  
	  basic_io_object(basic_io_object&& other)
	  {
	    ......
	  }
	  basic_io_object& operator=(basic_io_object&& other)
	  {
	    ......
	  }