---
id: 56
date: "2015-02-25"
title: "Put Your Controllers On a Diet With Service Objects"
---
When you use the Model-View-Controller pattern with Rails, you can be dragged into a state of only making models for your ActiveRecord objects. This can be bad if your controller needs to interact with two different models where either the logic doesn't belong in one, or it is simply too cumbersome to fit into a single model. Take for example user sign up. If a bunch of actions need to be fired off when the user signs up, then you might think to put them into a controller or the user model, or even worse, as ActiveRecord Callbacks. Another example is an AirBnB type website where the user makes a reservation on a listing. Sure you could put it in the listing model but now the listing is responsible for deleting itself and creating a reservation, violating the Single Responsibility Principle. Because the models are supposed to describe a single piece of logic, you can make another model that simply glues all of the models together. You can think of this model as a way to do everything on a higher level as opposed to getting down to the base of the application. This is the purpose of a service object.There is also a significant benefit to testing when using service objects. If you put all your logic in your controller, then you can only test it with an integration test, which must fire up a headless web browser, which you know is going to be slow. Because you are using a service object, you can rigorously test the model quickly, which will lead to wider test coverage, getting all of those edge cases, and it will make you want to test that file more.Now that we know we need to use this, how would we go about building this. In the case of the AirBnB style website, we would make a `ReserveListing` model which would be initialized with the two parties and the parameters, such as start date and end date.

    class ReserveListing
     def initialize(tenant, landlord, residence, start_time, end_time)
       @tenant = tenant
       @landlord = landlord
       @start_time = start_time
       @end_time = end_time
     end
    end

This is good so far, but that is quite a number of arguments that this service object depends on. We can cut this down easily by making a date range object and injecting it as an argument. This is called Dependency Injection, and the benefit is it forces us to separate the models even further so they are not too dependent on each other.

     class ReserveListing
     def initialize(tenant, landlord, residence, date_range)
       @tenant = tenant
       @landlord = landlord
       @residence = residence
       @start_time = date_range.start_time
       @end_time = date_range.end_time
     end
    end
     
This makes interacting with this class from the outside easier as all you need is the parties that are involved and some object that responds to `start_time` and `end_time`. If you think about how this class will be instantiated and used, it will have a very nice API.

    residence = Residence.find(params[:residence_id])
    landlord = residence.landlord
    date_range = DateRange.new(reservation_params)
    booking = ReserveListing.new(current_user, landlord, residence, date_range).book
    
You must make this `DateRange` class that is a container and date parser for `start_time` and `end_time`. Also, you don't want to put the logic of finding the landlord and residence inside of the service object because in this case it would be better to use Dependency Injection. Inside the `ReserveListing` class is where you must put the code to remove the listing at those times and create the reservation, probably using a transaction, with an added benefit of having any other actions you'd like fire off without using the appropriately hated ActiveRecord Callbacks. So use service objects and your controllers won't even need a personal trainer to stay in shape.