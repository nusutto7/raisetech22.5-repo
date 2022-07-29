#### テストファイル
`spec/localhost/sample_spec.rb`に以下のように記述。
```ruby
require 'spec_helper'

listen_port = 3000

describe package('nginx') do
  it { should be_installed }
end

describe package('unicorn') do
  it { should be_installed.by('gem').with_version('5.4.1') }
end

describe command('ruby -v') do
  its(:stdout) { should match /ruby 2\.6\.3/ }
end

describe command('ls -al /var/www/raisetech-live8-sample-app/config')do
  its(:stdout) { should match /credentials.yml.enc/}
end

describe command('ls /usr') do
  its(:exit_status) { should eq 0}
end

describe command('ls /foo') do
  its(:stderr) { should match /No such file or directory/ }
end

describe file('/etc/passwd') do
  it { should exist}
end

describe file('/var/www/raisetech-live8-sample-app/.gitignore') do
  its(:content) { should match /config\/master.key/ }
end


describe port(listen_port) do
  it { should be_listening }
end

describe command('curl http://127.0.0.1:#{listen_port}/_plugin/head/ -o /dev/null -w "%{http_code}\n" -s') do
  its(:stdout) { should match /^200$/ }
end

describe routing_table do
  it do
    should have_entry(
      :destination => '169.254.169.254',
      :interface   => 'eth0',
    )
  end
end
```

#### 実行結果
```
$ rake soec:localhost
/home/ec2-user/.rbenv/versions/2.6.3/bin/ruby -I/home/ec2-user/.rbenv/versions/2.6.3/lib/ruby/gems/2.6.0/gems/rspec-support-3.11.0/lib:/home/ec2-user/.rbenv/versions/2.6.3/lib/ruby/gems/2.6.0/gems/rspec-core-3.11.0/lib /home/ec2-user/.rbenv/versions/2.6.3/lib/ruby/gems/2.6.0/gems/rspec-core-3.11.0/exe/rspec --pattern spec/localhost/\*_spec.rb

Package "nginx"
  is expected to be installed

Package "unicorn"
  is expected to be installed by "gem" with version "5.4.1"

Command "ruby -v"
  stdout
    is expected to match /ruby 2\.6\.3/

Command "ls -al /var/www/raisetech-live8-sample-app/config"
  stdout
    is expected to match /credentials.yml.enc/

Command "ls /usr"
  exit_status
    is expected to eq 0

Command "ls /foo"
  stderr
    is expected to match /No such file or directory/

File "/etc/passwd"
  is expected to exist

File "/var/www/raisetech-live8-sample-app/.gitignore"
  content
    is expected to match /config\/master.key/

Port "3000"
  is expected to be listening

Command "curl http://127.0.0.1:#{listen_port}/_plugin/head/ -o /dev/null -w "%{http_code}\n" -s"
  stdout
    is expected to match /^200$/

Routing Table
  is expected to have entry {:destination=>"169.254.169.254", :interface=>"eth0"}

Finished in 0.41379 seconds (files took 0.38695 seconds to load)
11 examples, 0 failures
```

#### 雑感
テストコードと、ターミナルに投げられるコマンドの対応関係を分かる範囲で調べてみました。(一部分からず。。。)
https://github.com/tomohiro-tanazawa/TIL/blob/main/Serverspec/0729_serverspec_practice.md
テストコードがどんなコマンドに化けるのかを調べるのがとても楽しい。あと、Linuxの勉強にもなると感じた。
逆に、仕様書といえるだけのクオリティのテストを書くには、Linuxの知識が必須だと思った。
IGWまわりのテストを書きたかったが、知識不足で書けなかった。公式に載っているサンプルテストは一通り試してみたい。
