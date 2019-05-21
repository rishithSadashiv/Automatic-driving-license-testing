# Automatic-driving-license-testing

import RPi.GPIO as GPIO
import time
import Pyrebase
config={""}
firebase=pyrebase.initialize_app(config)
trigger_Ramp1=2
echo_Ramp1=3
trigger_Ramp2=17
echo_Ramp2=27
ir_pin1=10
ir_pin2=9
ultimatePin=18
switch1=26
switch2=19
test2_finish=5
GPIO.setmode(GPIO.BCM)
GPIO.setup(trigger_Ramp1,GPIO.OUT)
GPIO.setup(echo_Ramp1,GPIO.IN)
GPIO.setup(trigger_Ramp2,GPIO.OUT)
GPIO.setup(echo_Ramp2,GPIO.IN)
GPIO.setup(ir_pin1,GPIO.IN)
GPIO.setup(ir_pin2,GPIO.IN)
GPIO.setup(ultimatePin,GPIO.IN)
GPIO.setup(switch1,GPIO.IN)
GPIO.setup(switch2,GPIO.IN)
GPIO.setup(test2_finish,GPIO.IN)

def send_trigger_pulse(trigger_Ramp):
    GPIO.output(trigger_Ramp,True)
    time.sleep(0.0001)
    GPIO.output(trigger_Ramp,False)

def wait_for_echo(value,timeout,echo_Ramp):
    count=timeout
    while GPIO.input(echo_Ramp) != value and count > 0:
        count =count-1

def get_distance(trigger_Ramp,echo_Ramp):
    send_trigger_pulse(trigger_Ramp)
    wait_for_echo(True,10000,echo_Ramp)
    start=time.time()
    wait_for_echo(False,10000,echo_Ramp)
    finish=time.time()
    pulse_len=finish-start
    distance_cm=pulse_len/0.000058
    return (distance_cm)

def Ramp():
    points=100
    while True:
        a=18
        while a<19:
            time.sleep(1)
            a=get_distance(trigger_Ramp1,echo_Ramp1)
            print(a)
            if a< 10.8 and a>7.3:
                points=points-2
                print(points)
                time.sleep(2)
            elif a<= 7.3:
                points=points-4
                print(points)
            else:
                time.sleep(2)
                if a<10.8:
                    continue
                else:
                    if a>19:
                        print(points)
                        return points
        print(points)
        return points


def curve():
    crash=0
    points=100
    r=0
    start=time.time()
    while True:
        a=GPIO.input(switch1)
        b=GPIO.input(switch2)
        if a:
            while a:
                a=GPIO.input(switch1)
                time.sleep(0.2)
            points=points-10
            print(points)
            crash=crash+1
            print(crash)
            print(a)
            continue
        if b:
            while b:
                b=GPIO.input(switch2)
                time.sleep(0.2)
            points=points-10
            print(points)
            crash=crash+1
            print(crash)
            print(b)
            continue
        finish=time.time()
        tTime=int(finish-start)
        if test2_finish==1:
            return points
        if(tTime>=20):
            if crash==0:
	            points=0
            return points,crash


def overtake():
    while True:
        points=100
        a=get_distance(trigger_Ramp2,echo_Ramp2)
        time.sleep(1)
        print(a)
        temp=0
        while a< 25:
            a=get_distance(trigger_Ramp2,echo_Ramp2)
            temp+=1
            if a<7.2 or a>13.5 and a<25:
                print("too close or too far")
                points=points-10
                time.sleep(1)
                print(points)
            else:
                print("good")
                time.sleep(1)
                print(points)
        if temp>0:
            break
    return points



def parking():
     points=100
     c=0
     start=time.time()
     while True:
        finish=time.time()
        tTime=int(finish-start)
        print(tTime)
        if tTime<=20:
            a=GPIO.input(ir_pin1)
            b=GPIO.input(ir_pin2)
            if (a==0 or b==0 ):
                points=0
                print(points)
                time.sleep(1)
                return points
            u=GPIO.input(ultimatePin)
            if u==1:
                return points
        else:
            points=0
            return points
while True:
    data1=0		
    i=0
    l=[]

    for i in range(0,20):
    
        l.append(data.child("/").child("customer").child(i+1).child("status").get().val())
	        i=i+1
    while True:
        for j in l:
	        if(j==1):
		        name=data.child("/").child("customer").child(l.index(j)+1).child("cname").get().val()
                id=data.child("/").child("customer").child(l.index(j)+1).child("cid").get().val()
                value1 = Ramp()
                points,crash  = curve()
                value3 = overtake()
                value4 = parking()
                totalScore = value1+points+value3+value4
                if(totalScore>=350):
			        result='PASS!!!'
                else:
                    result='FAIL'
			
            			
			    data.child("/").child("customer").child(l.index(j)+1).child("result").set(result)
			    data.child("/").child("customer").child(l.index(j)+1).child("status").set(data1)
			    break
	    break
