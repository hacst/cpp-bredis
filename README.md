# bredis
Boost::ASIO low-lever redis client (connector)

[![Travis](https://img.shields.io/travis/basiliscos/cpp-bredis.svg)]()
[![license](https://img.shields.io/github/license/basiliscos/cpp-bredis.svg)]()

## Features

- header only
- low-level controls (i.e. you can cancel, or do you manual DNS-resolving before connection)
- unix domain sockets support
- synchronous & asynchronous interface

## Syncronous TCP-connection example

```cpp
#include <bredis/SyncConnection.hpp>
#include <boost/variant.hpp>
#include <boost/utility/string_ref.hpp>
...
namespace r = bredis;
namespace asio = boost::asio;

...
/* define used socket type */
using socket_t = asio::ip::tcp::socket;
...
/* establishing connection to redis is outside of bredis */
asio::ip::tcp::endpoint end_point(
    asio::ip::address::from_string("127.0.0.1"), port);
socket_t socket(io_service, end_point.protocol());
socket.connect(end_point);

/* buffer is not allocated inside bredis */ 
asio::streambuf rx_buff;
r::SyncConnection<socket_t> connection(std::move(socket));
/* get the result, boost::variant */ 
auto result = c.command("ping", rx_buff);
/* we know what the type is, safe to unpack to string_ref */
auto &reply_str = boost::get<r::string_holder_t>(result).str;
/* now we have a string copy, which is actually "PONG" */
std::string str(reply_str.cbegin(), reply_str.cend());
```

## Asyncronous TCP-connection example
```cpp
#include "bredis/AsyncConnection.hpp"
#include <boost/variant.hpp>
#include <boost/utility/string_ref.hpp>
...
namespace r = bredis;
namespace asio = boost::asio;
...
/* define used socket type */
using socket_t = asio::ip::tcp::socket;
...
/* establishing connection to redis is outside of bredis */
asio::ip::tcp::endpoint end_point(
    asio::ip::address::from_string("127.0.0.1"), port);
socket_t socket(io_service, end_point.protocol());
socket.connect(end_point);

r::AsyncConnection<socket_t> redis_connector(std::move(socket));
redis_connector.push_command("LLEN", "my-queue", 
                             [](const auto &error_code, r::some_result_t &&r) {
    int my_queue_size = boost::get<r::int_result_t>(r);
});

```

## Asyncronous unix domain sockets connection example
```cpp
#include "bredis/AsyncConnection.hpp"
#include <boost/variant.hpp>
#include <boost/utility/string_ref.hpp>
...
namespace r = bredis;
namespace asio = boost::asio;
...
/* define used socket type */
using socket_t = asio::local::stream_protocol::socket;
...
/* establishing connection to redis is outside of bredis */
asio::local::stream_protocol::endpoint end_point("/tmp/redis.socket");
socket_t socket(io_service, end_point.protocol());
socket.connect(end_point);

/* async interface remains the same as for TCP-connectinos */
r::AsyncConnection<socket_t> redis_connector(std::move(socket));
redis_connector.push_command("LLEN", "my-queue", 
                             [](const auto &error_code, r::some_result_t &&r) {
    int my_queue_size = boost::get<r::int_result_t>(r);
});
```

## API

### `some_result_t`

The `some_result_t` is `boost::variant` of the following types: 
- `string_holder_t`
- `error_holder_t`
- `nil_t`
- recursive array wrapper of `some_result_t` (`boost::recursive_wrapper<array_holder_t>`)

`some_result_t` and `error_holder_t` just have `str` member, which is basically `boost::string_ref`.

`nil_t` is obvious type to present `nil` redis result. 

`array_holder_t` has `elements` member, which is `std::vector` of `some_result_t`.


# License 

MIT

## See also
- https://github.com/Cylix/cpp_redis
- https://github.com/blackjack/booredis
- https://github.com/nekipelov/redisclient