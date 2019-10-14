---
id: 40
date: "2014-10-10"
title: "Mock external services with a VCR"
---
When creating an Rails App using TDD, sometimes you will come across the need for an external service, for example, using the Google Maps API. Over the course of testing, you will find that anything that accesses an external service, takes too long, doesn't work without internet connection, and adds to your request limit. This can be very problematic if you do programming offline, say on a train, as those tests that use external services will all fail. How to solve this problem and still get your tests to pass can sound complicated at first, but there's a gem for that.

We will actually be using two gems for this, one to block the tests from contacting anything but `localhost` and one to grab the response once from the external service and refer to the original response on every subsequent test.

The first gem is [WebMock](https://github.com/bblimke/webmock) to disable HTTP requests. To begin add to your `Gemfile`

    gem "webmock"

and run

    $ bundle install

At the top of `spec_helper.rb`, you will need to add

    require "webmock/rspec"

WebMock.disable_net_connect!(allow_localhost: true)

This inserts WebMock into your tests so now you can't access any external services and the tests will yell at you when you try with an error message like

    WebMock::NetConnectNotAllowedError:
    Real HTTP connections are disabled. Unregistered request: GET "http://maps.googleapis.com/maps/api/geocode/json?address=350%205th%20Avenue%20New%20York,%20NY%2010118&language=en&sensor=false" with headers {'Accept'=>'*/*', 'Accept-Encoding'=>'gzip;q=1.0,deflate;q=0.6,identity;q=0.3', 'User-Agent'=>'Ruby'}

 You can stub this request with the following snippet:

    stub_request(:get, "http://maps.googleapis.com/maps/api/geocode/json?address=350%205th%20Avenue%20New%20York,%20NY%2010118&language=en&sensor=false").
      with(:headers => {'Accept'=>'*/*', 'Accept-Encoding'=>'gzip;q=1.0,deflate;q=0.6,identity;q=0.3', 'User-Agent'=>'Ruby'}).
      to_return(:status => 200, :body => "", :headers => {})

    registered request stubs:

    stub_request(:get, "http://maps.googleapis.com/").
      with(:headers => {'Accept'=>'*/*', 'User-Agent'=>'Ruby'})

    ============================================================

This gem is cool that it tells you how to fix it, but you would have to type out exactly what you want to get from the external service, as it doesn't know what is supposed to be in the body. Wouldn't it be cool to just run the test once and all of the response gets copied for later usage. The [VCR](https://github.com/vcr/vcr) gem does just that. To get that working, in your `Gemfile`

    gem "vcr"

and run

    $ bundle install

Now we must configure VCR to stop WebMock once to record, and then stub out the request every subsequent time. To configure make a `vcr_setup.rb` file in `spec/support` and you will add the configuration details like

    require "vcr"

    VCR.configure do |c|
      c.cassette_library_dir = "vcr_cassettes"
      c.hook_into :webmock
    end

What this is doing is creating a directory called `vcr_cassettes` and creating a response YAML file for each unique request in that directory. Since it is a very modular gem, it allows you to interact with not just WebMock but other gems as well, so we need to tell it which gem we are using.

When you run the tests it will then tell you where it is failing and give you an idea of how to fix it, but the error message can seem a little unnecessary. All it really wants you to do is every time you are making a request to the same external service, just wrap it in a VCR block. For example,

    describe Location do
      context ".search" do
        it "returns a correct search result" do
          VCR.use_cassette("geolocate") do
            @location = create(:location)
          end

          expect(Location.search("Empire State Building")).to include(@location)
        end
      end
    end

The first time you run the test it will contact Google Maps and record the response in a YAML file in `vcr_cassettes/geolocate.yml` and will grab that response on every subsequent test where the name passed in to `.use_cassette` is the same. One thing to note is when you are running an integration test, you only need to wrap whatever creates the location in the `VCR.use_cassette` block, so with Capybara

    ...
    fill_in "Name", with: location.name

    VCR.use_cassette("geolocate") do
      click_button "Add Location"
    end
    ...

Your tests will now pass on a plane or a train, almost like you have a cassette player with you.