---
layout: post
title: Watering System
tags: watering system arduino ruby serialport
---

Vacation time! Finally my wife and I can go on vacation. The only problem that occurs every time, are the plants that need to be watered. So watering the plants is fairly simple but can be made a real geek project.

This is where it all started one year ago. I build the first prototype of the watering system. First of all we need something where we get the water from. I chose a big waterproof plastic box. 

<img src="/images/posts/2013-08-18-plastic-box.jpg" alt="Plastic Box" width="70%"/>

This box is basically my water tank. To get the water outside the tank we need a pump. 

<img src="/images/posts/2013-08-18-pump.jpg" alt="Pump" width="70%"/>

The pump is the most cheapest pump I found in a local shop (12.99 Euro). I guess its main purpose is to circulate water in an aquarium. Next i searched for pipes with little drop out modules. Some of them are configurable. 

<img src="/images/posts/2013-08-18-drop-out.jpg" alt="Drop out" width="70%"/>

The configuration basically lets more water on the bigger plants. Because the pipes of that system are different than the one that i can plug into the pump I bought a transparent pipe that can connect both the pipe and the drop outs.

The pump basically starts working if you plug it into the alternating current. In order to controll it I can use a simple remote control ac switch. This switch uses a simple RC transmitter and receiver combination with a very simple protocol. Basically it would be easyer to use a simple timer plug, but this would be to simple and i wanted something more geeky. Therefore I hooked up the remote that came with the RC switch to my ardurino.

<img src="/images/posts/2013-08-18-arduino-rc.jpg" alt="Arduino RC" width="70%"/>

To use the arduino to send those RC switches commands I basically used an already existing library. The library supports the full featureset of the protocol and works awesomely good, it is called [RCSwitch](https://code.google.com/p/rc-switch/). To control the arduino i created a simple command protocol ([SCProtocol](https://github.com/threez/SCProtocol)) library. 

{% highlight cpp %}
#include <SCProtocol.h>
#include <RCSwitch.h>

RCSwitch rcSwitchControl = RCSwitch();
SCProtocol scProtocol = SCProtocol();

void checkCallback(char* data) {
  Serial.println("OK HomeControl Version: 0.0.1");
}

/* Turns the switch on.
 * @param data The data used to switch are in this format
 *             "11110A+". The first five chars are the group
 *             of the switch the next is the switch char
 *             followed by a plus (turn on) or a minus (turn of).
 */
void rcSwitchControlCallback(char* data) {
  // Parse the switch number and if it should be turned on
  byte switchNr = (data[5] - 'A') + 1;
  boolean on = data[6] == '+';
  
  // remove everything but the bit pattern
  data[5] = '\0';
  
  // do the magic
  if (on) rcSwitchControl.switchOn(data, switchNr);
  else    rcSwitchControl.switchOff(data, switchNr);
  
  // print result
  Serial.print("OK Switched ");
  Serial.print(data);
  Serial.print(" (");
  Serial.print(switchNr);
  Serial.print(") to ");
  Serial.println(on ? "ON" : "OFF");
}

void setup() {
  rcSwitchControl.enableTransmit(8);
  Serial.begin(9600);
  scProtocol.attach('C', checkCallback, 0);
  scProtocol.attach('S', rcSwitchControlCallback, 7);
}

void loop() {
  while(Serial.available()) scProtocol.process(Serial.read());
}
{% endhighlight %}

The arduino is pluged into my homeserver using USB. Background: At home i already have a homeserver that is always on and connected to the internet. This homeserver is the controller of the watering system. Then i use the following ruby library i wrote to connect to the arduino:

{% highlight ruby %}
require "rubygems"
require "serialport"

class HomeControl
  class Error < StandardError; end 

  def initialize(path)
    @sp = SerialPort.new path, 9600, 8, 1, SerialPort::NONE
    sleep 2
    @sp.read_timeout = 1500
    cmd "C" 
  end 

  def self.open(path, &block)
    ab = new(path)
    ab.session(&block)
    ab.close
  end 

  def switch(group, states = {}) 
    states.each do |name, state|
      cmd "S#{group}#{name}#{state == :on ? '+' : '-'}"
    end 
  end 

  def cmd(command)
    STDERR.puts "-> Sending '#{command}'"
    @sp.write command
    result = @sp.readline
    STDERR.puts "-> Received '#{result.chomp}'"
    raise Error, result if !ok? result
  end 

  def ok?(str)
    str =~ /^OK/
  end 

  def session(&block)
    block.call(self)
  end 

  def close
    @sp.close
  end 
end

{% endhighlight %}

With this little library it is very simple to write then the watering script itself. Before we need to check the id (group and device number) of the RC switch. 

<img src="/images/posts/2013-08-18-rc-switch.jpg" alt="rc-switch" width="70%"/>

The following script turns on the rc switch for five minutes. I trigger the watering every 3th day using crontab.

{% highlight ruby %}
require './home_control'

group = '11111'
device = 'B' 
duration = 5 * 60 # 5 min

HomeControl.open('/dev/ttyACM0') do |controller|
  controller.switch group, device => :on 
  sleep duration
  controller.switch group, device => :off
end
{% endhighlight %}

Basically this system is very extensible an can be used to create all sorts of remote controlled systems based on those RC switches. To sum up this was a fun project and I learned a lot here about arduino.
