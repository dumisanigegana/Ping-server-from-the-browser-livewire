# Measuring the network ping time Client and the server on Livewire.

###### This recipe guides through the implementation of ICMP ping testing a network ICMP ping on a server render application, to find the round trip time from the server to the client.  

## Problem

You might be developing a time-sensitive game and you need to factor in the network latency time, in your code. On the other hand, you are concerned about the performance of your web application and you want to monitor the network latency, knowing that the ping time, also referred to as the round trip time (RTT),  as we develop further or work on improving it. 
Consider a situation where you are working on a server-rendered web application and you have total visibility of the RTT test results. The fact is that when you see your key metrics in your dashboard, you intuitively start improving the results.

How possible it is to come up with means of tracking the round trip time, between a client and servers, within a server-rendered application?

## Solution

I created a Livewire component that runs a ping between the web server and the client’s browser, and this blog will show you how simple it is to do it. The test is initiated from the browser, by clicking a button, then the server records the current time, and then it sends a ping to the IP address of the client. Upon receiving the response from the client, the current time is recorded and the ping time is computed by finding the difference between the two recorded times. Finally, the server sends it to the browser computes the ping, and returns the ping time to the browser. 

Here is a demostration of what we well create in the next 20 minutes:
[![The ping time component demo](https://github.com/dumisanigegana/Ping-server-from-the-browser-livewire/blob/12a6b40e463ff311dfe616528d7fb131beaa2810/using-livewire-to-test-ping-time-demo.gif)](# "The ping time component demo")

###### Prerequisites. 
To proceed with this tutorial, be sure that you have a Laravel and Livewire installed and configured. Note, for designing, I used Tailwind CSS, however you may replace it according to your preferences. 

###### Generate the Component files.
We begin by creating the component files, using the artisan command, as follows:

`php artisan make:livewire ShowPings` 

This generates two files as follows: 
1. The `app/Http/Livewire/ShowPosts.php` is a class that holds the logic of our ping test component. 
2. The `resources/views/livewire/show-posts.blade.php` for designing the view layout of the component and for initiating server actions.

###### The ShowPings class
In the `ShowPings.php,` let’s begin by declaring the public properties, namely the` $pingTimes`, `$clientIP`, and `$ping`. So we insert them as follows:
```php
 public $pingTimes = array(), $clientIP, $ping;
 ```
The $pingTimes is for holding the round trip times in an array, the $clientIP is for holding the IP address of the client, and the $ping property will store the state of the component.
By default the render() function has been created for us. Lets go ahead and create the mount(), pingInit() and reset() functions as follows:

```php
class ShowPings extends Component
{
    public $pingTimes = array(), $clientIP, $ping;

    public function mount()
    {
      $this->clientIP = \Request::ip(); // Get client Ip address.
    }

    public function pingInit()
    {
      $exec_starttime = microtime(true); //Record the start time.
      exec("ping -c 1 $this->clientIP");   // : Ping the client IP once.
      $results = microtime(true) - $exec_starttime;  // Computes RTT.
      array_push($this->pingTimes, round($results * 1000)); //Converts RTT an integer in ms.
    }

     public function resetPingTimes()
    { 
      if($this->ping)
      {
        $this->pingTimes = array(); // Reset the RTTs.
      }
    }

    public function render()
    {
      return view('livewire.show-pings');
    }

}
```
The mount function simply gets the IP address of the client node and mounts it to the $clientIP property.  
In `pingInit()` the first line records the starting time on the pinging and the second line executes the ping process of the client IP address. Note we use `–c 1` to limit ping rounds to one instead of 4, on Window us `–n 1`. The third line records the final time and computes the ping time in seconds. The Fourth line converts the ping time from seconds to milliseconds, rounds it to the nearest integer, and then pushes it to the pingTimes property. The last line delays exiting the function in turn delaying the activation of the ping button in show-posts.blade.php. 
Based on the $ping state, the `reset()` function simply clears the ping time in the pingTimes property, resetting it to an empty array.

######The View
Suppose we want our ShowPings component to appear on the default homepage of Laravel, then in the welcome.blade.php insert this line: `<livewire:show-pings />`, where you want the component to be rendered. For instance:
```html
      <div class="mt-8 bg-white dark:bg-gray-800 items-top shadow sm:rounded-lg w-full overflow-y-auto max-h-96">
         <livewire:show-pings />           
      </div>
```
Note, remember to include the Livewire assets in welcome.blade.php.

 Now let’s dive into our component blade file where we design our component layout. Here need a checkbox to start and stop the ping process, a polling component to run continuously the pingInit function, and an element for displaying the ping times.

```html
<div >
   <p class="text-2xl text-blue-500 py-4 font-bold text-center w-full">Click to ping the server </p>
   <div class="flex justify-center w-full mb-2">
      <div class="form-check">
         <input class="form-check-input appearance-none h-8 w-24 border-2 border-green-300 rounded-lg bg-gray-100 checked:bg-red-400 checked:border-red-500 mt-1 align-top bg-no-repeat bg-center bg-contain float-left mr-2 cursor-pointer" 
		type="checkbox"  value="1" wire:model="ping" wire:click="resetPingTimes()">
       </div>
    @if($ping)
    <div wire:poll.1s="pingInit()"> </div>
    @endif
   </div>
      @if (count($pingTimes)>0)
      @foreach($pingTimes as $time)
   <div class="flex justify-center text-center p">
      <div class="inline-flex justify-center text-center">
         <span class=" mr-12">{{ $loop->iteration }} : </span>
         <span class="font-bold mr-2">{{ $time }}</span>
         <span> miliseconds</span>
      </div>
   </div>
      @endforeach
      @endif
</div>
```
Take note of the `wire:` directives in the checkbox input tag. The `wire:model=”ping"` binds the checkbox value to the $ping property, in the ShowPosts  Class. When the checkbox has clicked the directive` wire:click =" reset()"`  instructs the browser to dispatch a click event, and it passes the action to be executed, in this case, the `reset()`.

Next, we used the `@if` derivative conditionally display the polling component, based on the value of the `$ping` state. This polling component continuously executes the `pingInit()` function. The `.1s` forces the execution to to be fired once after every second.

Finally, we display the results in the file show-posts.blade.php, by looping through the `$pingTimes` array.
We have completed the task and I hope you enjoyed the simplicity of Livewire. Basically thi is the flawchart if the of the componet we just created:
Here is the logic flow.
[![The ping time component flowchart](https://github.com/dumisanigegana/Ping-server-from-the-browser-livewire/blob/12a6b40e463ff311dfe616528d7fb131beaa2810/using-livewire-to-test-ping-time-flowchat.png)](# "The ping time component flowchart")


## Discussion

It is amazing how Livewire empowers developers with the ability to implement efficient real-time and post-event analytics, from either the client or server. This blog may help in developing tools for troubleshooting and monitoring networks to multiple remote sites, and real-time data visualization may be incorporated to display the returned values.

-- 
