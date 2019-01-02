# NAME

Test::Mock::Net::Server::Mail - mock SMTP server for use in tests

# VERSION

version 1.01

# SYNOPSIS

In a test:

    use Test::More;
    use Test::Mock::Net::Server::Mail;

    use_ok(Net::YourClient);

    my $s = Test::Mock::Net::Server::Mail->new;
    $s->start_ok;

    my $c = Net::YourClient->new(
      host => $s->bind_address,
      port => $s->port,
    );
    # check...

    $s->stop_ok;

# DESCRIPTION

Test::Mock::Net::Server::Mail is a mock SMTP server based on Net::Server::Mail.
If could be used in unit tests to check SMTP clients.

It will accept all MAIL FROM and RCPT TO commands except they start
with 'bad' in the user or domain part.
And it will accept all mail except mail containing the string 'bad mail content'.

If a different behaviour is need a subclass could be used to overwrite process\_&lt;verb> methods.

# LOGGING

If the logging option is enabled (by default) the mock server will log
received commands in a temporary log file. The content of this log file
can be inspected with the methods next\_log() or tested with next\_log\_ok().

    # setup server($s) and client($c)...

    $c->ehlo('localhost');
    $s->next_log;
    # {"verb" => "EHLO","params" => "localhost"}
    
    $c->mail_from('user@domain.tld');
    $s->next_log_ok('MAIL', 'user@domain.tld, 'server received MAIL cmd');
    
    $c->rcpt_to('targetuser@targetdomain.tld');
    $s->next_log_ok('RCPT', qr/target/, 'server received RCPT cmd');

    # shutdown...

# ATTRIBUTES

## bind\_address (default: "127.0.0.1")

The address to bind to.

## start\_port (default: random port > 50000)

First port number to try when searching for a free port.

## support\_8bitmime (default: 1)

Load 8BITMIME extension?

## support\_pipelining (default: 1)

Load PIPELINING extension?

## support\_starttls (default: 1)

Load STARTTLS extension?

## logging (default: 1)

Log commands received by the server.

## mock\_verbs (ArrayRef)

Which verbs the server should add mockup to.

By default:

    qw(
      EHLO
      HELO
      MAIL
      RCPT
      DATA
      QUIT
    )

## next\_log

Reads one log from the servers log and returns a hashref.

Example:

    {"verb"=>"EHLO","params"=>"localhost"}

## next\_log\_ok($verb, $expect, $text)

Will read a log using next\_log() and test it.

The logs 'verb' must exactly match $verb.

The logs 'params' are checked against $expected. It must be a
string,regexp or undef.

Examples:

    $s->next_log_ok('EHLO', 'localhost', 'server received EHLO command');
    $s->next_log_ok('MAIL', 'gooduser@gooddomain', 'server received MAIL command');
    $s->next_log_ok('RCPT', 'gooduser@gooddomain', 'server received RCPT command');
    $s->next_log_ok('DATA', qr/bad mail content/, 'server received DATA command');
    $s->next_log_ok('QUIT', undef, 'server received QUIT command');

# METHODS

## port

Retrieve the port of the running mock server.

## pid

Retrieve the process id of the running mock server.

## before\_process( $smtp )

Overwrite this method in a subclass if you need to register additional
command callbacks via Net::Server::Mail.

Net::Server::Mail object is passed via $smtp.

## process\_ehlo( $session, $name )

Will refuse EHLO names containing the string 'bad'
otherwise will accept any EHLO.

## process\_mail( $session, $addr )

Will accept all senders except senders where
user or domain starts with 'bad'.

## process\_rcpt( $session, $addr )

Will accept all reciepients except recipients where
user or domain starts with 'bad'.

## process\_data( $session, \\$data )

Overwrite on of this methods in a subclass if you need to
implement your own handler.

## main\_loop

Start main loop.

Will accept connections forever and will never return.

## start

Start mock server in background (fork).

After the server is started $obj->port and $obj->pid will be set.

## start\_ok( $msg )

Start the mock server and return a test result.

## stop

Stop mock smtp server.

## stop\_ok( $msg )

Stop the mock server and return a test result.

# AUTHOR

Markus Benning <ich@markusbenning.de>

# COPYRIGHT AND LICENSE

This software is copyright (c) 2015 by Markus Benning <ich@markusbenning.de>.

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.
