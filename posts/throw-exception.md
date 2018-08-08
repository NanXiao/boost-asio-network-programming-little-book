# Throw exception

`Boost.Asio` functions may throw `boost::system::system_error` exception. Take `resolve` as an example:  

	results_type resolve(BOOST_ASIO_STRING_VIEW_PARAM host,
		BOOST_ASIO_STRING_VIEW_PARAM service, resolver_base::flags resolve_flags)
	{
	  boost::system::error_code ec;
	  ......
	  boost::asio::detail::throw_error(ec, "resolve");
	  return r;
	}

There are two overloads of `boost::asio::detail::throw_error` functions:  

	inline void throw_error(const boost::system::error_code& err)
	{
	  if (err)
	    do_throw_error(err);
	}
	
	inline void throw_error(const boost::system::error_code& err,
	    const char* location)
	{
	  if (err)
	    do_throw_error(err, location);
	}
The differences of these two functions is just including "location" ("`resolve`" string in our example) or not. Accordingly, `do_throw_error` also have two overloads, I just take one as an example:  

	void do_throw_error(const boost::system::error_code& err, const char* location)
	{
	  boost::system::system_error e(err, location);
	  boost::asio::detail::throw_exception(e);
	}

`boost::system::system_error` derives from `std::runtime_error`:  

	class BOOST_SYMBOL_VISIBLE system_error : public std::runtime_error
	{
	......
	public:
	      system_error( error_code ec )
	          : std::runtime_error(""), m_error_code(ec) {}
	
	      system_error( error_code ec, const std::string & what_arg )
	          : std::runtime_error(what_arg), m_error_code(ec) {}
	......
	      const error_code &  code() const BOOST_NOEXCEPT_OR_NOTHROW { return m_error_code; }
	      const char *        what() const BOOST_NOEXCEPT_OR_NOTHROW;
	......
	}

`what()` member function returns the detailed information of exception.