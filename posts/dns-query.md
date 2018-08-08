# DNS query

The `resolver` class is used to do `DNS` query, i.e., convert a host+service to `IP`+port. Take `boost::asio::ip::tcp::resolver` as an example:  

	#include <boost/asio.hpp>
	#include <iostream>
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	
	        boost::asio::ip::tcp::resolver resolver{io_context};
	        boost::asio::ip::tcp::resolver::results_type endpoints =
	                resolver.resolve("google.com", "https");
	
	        for (auto it = endpoints.cbegin(); it != endpoints.cend(); it++)
	        {
	            boost::asio::ip::tcp::endpoint endpoint = *it;
	            std::cout << endpoint << '\n';
	        }
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << '\n';
	    }
	
	    return 0;
	}

The running result is:   

	74.125.24.101:443
	74.125.24.139:443
	74.125.24.138:443
	74.125.24.102:443
	74.125.24.100:443
	74.125.24.113:443

The element of `boost::asio::ip::tcp::resolver::results_type`'s every iteration is `basic_resolver_entry`:  

	template <typename InternetProtocol>
	class basic_resolver_entry
	{
	......
	public:
	  /// The protocol type associated with the endpoint entry.
	  typedef InternetProtocol protocol_type;
	
	  /// The endpoint type associated with the endpoint entry.
	  typedef typename InternetProtocol::endpoint endpoint_type;
	......
	  /// Convert to the endpoint associated with the entry.
	  operator endpoint_type() const
	  {
	    return endpoint_;
	  }
	......
	}
Since it has `endpoint_type() ` operator, it can be converted to endpoint directly:  

	boost::asio::ip::tcp::endpoint endpoint = *it;
