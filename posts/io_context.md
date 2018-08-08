# io_context

Like traditional `Unix` network programming, `Boost.Asio`also has "socket" concept, but that's  not enough, an `io_context` object (`io_service` class is deprecated now) is needed to communicate with Operating System's `I/O` services. The architecture is like following:  
![image](https://raw.githubusercontent.com/NanXiao/boost-asio-network-programming-little-book/master/images/architecture.jpg) 

`io_context` derives from `execution_context`:  

	class io_context
	  : public execution_context
	{
	......
	}
While `execution_context` derives from `noncopyable`:  

	class execution_context
	  : private noncopyable
	{
	......
	}

Check `noncopyable` class definition:

	class noncopyable
	{
	protected:
	  noncopyable() {}
	  ~noncopyable() {}
	private:
	  noncopyable(const noncopyable&);
	  const noncopyable& operator=(const noncopyable&);
	};

It means the `io_context` object can't be copy constructed/copy assignment/move constructed/move assignment, So during initialization of socket, i.e., associate socket with `io_context`, the `io_context` should be passed as a reference. E.g.:  

	template <typename Protocol
	    BOOST_ASIO_SVC_TPARAM_DEF1(= datagram_socket_service<Protocol>)>
	class basic_datagram_socket
	  : public basic_socket<Protocol BOOST_ASIO_SVC_TARG>
	{
	public:
	......
	  explicit basic_datagram_socket(boost::asio::io_context& io_context)
	    : basic_socket<Protocol BOOST_ASIO_SVC_TARG>(io_context)
	  {
	  }
	......
	}