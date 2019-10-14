---
id: 9
date: "2014-06-30"
title: "Designing a Flash Card App in Sinatra"
---
During the first week of Metis, we were tasked to build a flash card game playable from the command line. We kept adding features, and what started out as a one deck flash card game, turned into a game that read the contents of a separate text file and entered that into the game each time it was played. The second week, we learned all about SQL, Sinatra, and Active Record, so for the weekend project we were tasked to create the flash card game for the web. As I was thinking about how to create this game, I thought the best design pattern would be to use a Model-View-Controller pattern, or MVC, which separates a lot of the code into distinct files. Being Sinatra and not Ruby on Rails, makes for trivial things to be potential mistakes, such as creating a database or creating a form, but it does illustrate fundamentals which do a great job of uncovering how everything works.

Specifically, you need to be able to perform the CRUD operations on the cards and decks, but you also need to be able to play the game, which can be implemented in a variety of ways. The way I used was I created a dummy route which would redirect the user to a random card and then be prompted for an input.

    get "/decks/:id/cards/random" do
      deck = Deck.find(params[:id])
      cards = Card.where(deck_id: deck.id)
      card = cards.sample
      redirect to("/decks/#{deck.id}/cards/#{card.id}")
    end
 
    get "/decks/:id/cards/:card_id"do
      @card = Card.find(params[:card_id])
      erb :front
    end
 
    post "/decks/:id/cards/:card_id/back" do
      card = Card.find(params[:card_id])
      input = params[:card][:back]
      if input.downcase == card.back.downcase
        redirect to("/decks/#{card.deck_id}/cards/#{card.id}/right")
      else
        redirect to("/decks/#{card.deck_id}/cards/#{card.id}/wrong")
      end
    end

For example, what this does is it creates a route called “/decks/1/cards/random” and creates an array of cards that have the FOREIGN KEY the same as the `deck.id`, which would be “1”. It then picks one card by using `#sample`. Then it redirects to the front of the card it picked. The reason for this roundabout way is to hide the next card from the user.

The next thing this does render `views/front.erb`. On the front.erb page, there is the contents of the front of the card along with an input box that uses a POST method.

Finally, the post method receives the input from the front.erb page, and checks if that input is the same as the back of the card and redirects to the appropriate page. A thing to note is the POST method does not necessarily have to save anything so you can use it as an input processor. You can view the source code on [Github](https://github.com/CharlesMassry/flash_card_sinatra).