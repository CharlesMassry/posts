---
id: 29
date: "2014-08-22"
title: "Testing Paperclip with RSpec, Capybara, and Factory Girl"
---
Recently I've been working on a project that allows users to upload photos or videos with the twist of using the same form and determining file type on the server. I decided to go this route because of the positive user experience a form that tells you to add either media type gives you. Regardless, manually testing the form can be a pain because it must be a photo or a video and the form can't be blank. This can involve a lot of work to test manually, between the clicking of the form, and clicking through to the path to the file. Enter feature specs.

    require "rails_helper"

    feature "Media creation" do
      before(:each) do
        @team = create(:team)
        @user = create(:user, team: @team)
        @play = create(:play, team: @team)
        sign_in(@user)
        visit team_play_path(@team, @play)
        click_link "Add photo or video"
      end

      scenario "I can add a photo to plays" do
        medium = build(:photo_medium)

        attach_file "File", "spec/asset_specs/photos/photo.jpg"
        fill_in "Caption", with: medium.caption
        click_button "Add photo or video"

        expect(page).to have_selector("img")
      end

      scenario "I can add a video to plays" do
        medium = build(:video_medium)

        attach_file "File", "spec/asset_specs/videos/video.MOV"
        fill_in "Caption", with: medium.caption
        click_button "Add photo or video"

        expect(page).to have_selector("video")
      end

      scenario "I cannot submit a blank file field" do
        click_button "Add photo or video"

        expect(page).to have_selector("form")
        expect(page).to have_css("#new_medium")
      end

      scenario "I cannot submit an invalid file type" do
        attach_file "File", "spec/asset_specs/fail/erd.pdf"
        fill_in "Caption", with: "should not work"
        click_button "Add photo or video"

        expect(page).to have_selector("form")
        expect(page).to have_css("#new_medium")
      end
    end  

What is happening here is before each test is run, a team, user, and play are all created and the user is then signed in and visits the `plays#show` page. Each test is then run, testing the 4 different possibilities of photo, video, wrong file type, and blank field. The main benefit of using [RSpec](http://rspec.info/) and [Capybara](http://jnicklas.github.io/capybara/) is that this reads almost like pure English. The RSpec sections are the end of each test that compares the expectations with the result and the Capybara sections are the beginning of each test that describes the interaction with a web browser. [Factory Girl](https://github.com/thoughtbot/factory_girl), (the Ruby Gem not the 2006 movie starring Sienna Miller) creates the data in the test database before each test is run with data referenced from a separate file. Now you can make all the changes to the site that you want and you'll know if they'll work now, and in the future.
