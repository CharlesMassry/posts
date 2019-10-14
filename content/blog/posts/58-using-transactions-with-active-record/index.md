---
id: 58
date: "2015-03-06"
title: "Using Transactions with Active Record"
---
A transaction is a database feature where you can create, update, or delete multiple records from a database and if just one of them fails, everything rolls back. For example, Postgres would roll everything back if there was some type of failure. ActiveRecord also provides support for this by allowing you to rollback if it is told to rollback. \n \n

In a previous [post](http://www.charliemassry.com/posts/56), I described why you would want to use service objects to encapsulate controller logic using an AirBnB style example application. Inside of this particular service object is exactly where you would want to use a transaction.


    class ReserveListing
      def initialize(tenant, landlord, residence, date_range)
        @tenant = tenant
        @landlord = landlord
        @residence = residence
        @start_time = date_range.start_time
        @end_time = date_range.end_time
      end
    
      def book
        ActiveRecord::Base.transaction do
          if residence_unavailable?
            raise ActiveRecord::Rollback
          end
          remove_time_from_residence
          create_reservation
        end
      end
    end
  
When a user reserves a residence, the model is responsible for checking if it is available, and if it isn't, rollback the transaction. You can imagine that these very broad methods that the service object is using can be really complex, especially for a controller. Also, it wouldn't fit in either of those ActiveRecord models because each model would become dependent on the other. with Another thing to note is in the above example, the conditional `if residence_unavailable?` method can trigger this `raise ActiveRecord::Rollback` exception. Normally you don't want to explicitly raise an exception in your code, but whenever you `raise ActiveRecord::Rollback` within a transaction, everything is cancelled and you can then handle what happens next.

Next, you may have to retrieve other information from this particular service object, such as sending out notifications. When you do this, you will need access to the reservation in this case. An easy way to gain access is to have a getter method and return self in the transaction.

    class ReserveListing
      attr_reader :reservation
      ...
      def book
        ActiveRecord::Base.transaction do
          if residence_unavailable?
            raise ActiveRecord::Rollback
          end
          remove_time_from_residence
          create_reservation
          self
        end
      end
    end

Now, inside of the controller action, you can then handle that reservation and pass it off to the `Notification` model, which is another service object to handle notifications.

    class ReserveListing
      ...
      def send_notifications
        Notification.new(self).broadcast
        # code that sends out notifications (email, text message, etc.)
      end
      ...
    end


Because you are returning self from `#book`, you can then call `#send_notifications` easily from the object.

One thing to note about transactions is that if it fails, it returns `nil`, which is actually perfect for this particular pattern.


    class ReservationsController < ApplicationController
      def create
        ...
        reserve_listing = ReserveListing.new(
          tenant,
          landlord,
          residence,
          date_range
        ) || NullReserveListing.new
        ...
      end
    end

Here we are using the Null Object Pattern, which I discussed in a previous [post](http://www.charliemassry.com/posts/27). All you have to do is implement the public methods that get triggered on `ReserveListing` which will be very simple as you will just return nil.

    class NullReserveListing
      def reservation
      end
    end

Now when you go to handle the redirect or response in `reservations#create` you can check the truthiness of the reservation getter method on whatever type of object `reserve_listing` is and this will tell you if your transaction succeeded or failed.


    class ReservationsController < ApplicationController
      def create
        ...
        if reserve_listing.reservation
          reserve_listing.send_notifications
          render "success"
        else
          render "error"
        end
      end
    end

Working with transactions can be difficult at first, but with a little work, you can leverage ActiveRecord's behavior to make your code very easy to come back to.