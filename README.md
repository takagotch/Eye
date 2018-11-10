### Eye
---
https://github.com/kostya/eye

```
gem install eye

eye load examples/test.eye
eye 1 examples/
eye 1 examples/*.rb
eye 1 CONF -f
/etc/eye.conf
~/.eyeconfig

eye info
eye i -j

eye restart all
eye r test
eye r samples
eye r sample1
eye sample*
eye r test:samples
eye r test:sample::sample1
eye r test:samples:sample*
eye r test:*sample*

eye check examples/test.eye
eye explain examples/test.eye

eye trace
eye t test
eye t sample

eye quit
eye a -s

eye watch
eye history
eye xinfo
eye x -c
```

```ruby
Eye.load('./eye/*.rb')
Eye.config do
  logger '/tmp/eye.log'
end
Eye.application 'test' do
  # except `env`, which merging down
  # uid "user_name"
  # gid "group_name"
  working_dir File.expand_path(File.join(File.dirname(__FILE__), %w[ processes ]))
  stdall 'trash.log'
  env 'APP_ENV' => 'production'
  trigger :flapping, times: 10, within: 1.minutes, retry_in: 10.minutes
  check :cpu, every: 10.seconds, below: 100, times: 3
  group 'sample' do
    chain grace: 5.seconds
    process :sample1 do
      pid_file '1.pid'
      start_command 'ruby ./smaple.rb'
      deamonize true
      stdall 'sample1.log'
      check :cpu, below: 30, times: [3, 5]
    end
    process :sample2 do
      pid_file '2.pid'
      start_command 'ruby ./sample.rb -d --pid 2.pid --log sample2.log'
      stop_command 'kill -9 {PID}'
      check :memory, every: 20.seconds, below: 300.megabytes, times: 3
    end
  end
  process :forking do
    pid_file 'forking.pid'
    start_command 'ruby ./forking.rb start'
    stop_command 'ruby forking.rb stop'
    stdall 'forking.log'
    start_timeout 10.seconds
    stop_timeout 5.seconds
    monitor_children do
      restart_command 'kill -2 {PID}'
      check :memory, below: 300.megabytes, times: 3
    end
  end
  process :event_machine do
    pid_file 'em.pid'
    start_command 'ruby em.rb'
    stdout 'em.log'
    daemonize true
    stop_signals [:QUIT, 2.seconds, :KILL]
    check :socket, addr: 'tcp://127.0.0.1:33221', every: 10.seconds, times: 2,
                  timeout: 1.second, send_data: 'ping', expect_data: /pong/
  end
  process :thin do
    pid_file 'thin.pid'
    start_command 'bundle exec thin start -R thin.rb -p 33233 -d -l thin.log -P thin.pid'
    stop_signals [:QUIT, 2.seconds, :TERM, 1.seconds, :KILL]
    check :http, url: 'http://127.0.0.1:33233/hello', pattern: /World/,
                 every: 5.seconds, times: [2, 3], timeout: 1.second
  end
end
```

```
```
