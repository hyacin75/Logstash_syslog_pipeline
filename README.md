It took me a while to pull all this together and tweak it just right, so I figured I'd put it online in case anyone else ends up in the same boat.

Getting rsyslog working with ELK is easy - it's just very hard to find the instructions on *how*!

In a multi-node setup on Ubuntu (and probably Debian - and maybe the same for a single node setup) drop the following in /etc/logstash/conf.d/ -

input {
  tcp {
    port => 5000
    type => syslog
  }
  udp {
    port => 5000
    type => syslog
  }
}

filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "(<%{POSINT:syslog_pri}>)?%{SYSLOGTIMESTAMP:syslog_timestamp} ?(%{SYSLOGHOST:syslog_hostname})? %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    syslog_pri {
      syslog_pri_field_name => "syslog_pri"
    }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}

output {
  elasticsearch { hosts => ["192.168.0.101:9200", "192.168.0.102:9200", "192.168.0.103:9200"] }
}



You can use whatever port you'd like, and obviously update the Elasticsearch IP(s) to what is right for your setup.

With this configuration my Logstash was able to accept and parse without error, rsyslog messages from my Ubuntu and Debian hosts, as well as my Asus router, Netgear switch, and Synology NAS.  It also assigns the correct facility and severity to the messages with this configuration.  I hope this saves someone else the days of searching and piecing together I had to do to get this working!!
