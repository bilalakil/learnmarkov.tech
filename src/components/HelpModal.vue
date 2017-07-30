<template>
    <div class="modal fade" id="helpModal" tabindex="-1" role="dialog" aria-labelledby="helpModalLabel">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h4 class="modal-title" id="helpModalLabel">About</h4>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
                </div>
                <div class="modal-body">
                  <p>
                    LearnMarkov.tech is an introductory-level resource allowing learners to experiment with character-based Markov Chains and generate creative, entertaining business names in the process.
                  </p>

                  <p>
                    The application uses a list of Australian Business names, which we first break into N-grams (the character strings of a specified length). We then generate a new, fake business name by starting off with a randomly chosen N-gram, and proceeding to attach N-grams to it based on their probability of appearing in that sequence (from the source dataset).
                  </p>

                  <p>
                    There are two modes available in the application:
                  </p>

                  <ol>
                    <li>
                      Parser: You'll see the Parser mode upon first opening the website. The Parser mode gives you control over how we break the list of business names into specifically sized N-grams using the 'Left' and 'Right' size selectors. You can see a preview of how we're breaking the words up, and control the speed that it plays at.
                    </li>
                    <li>
                      Name Generation: Once you've chosen your settings and moved to Name Generation, we parse the business name dataset using the chosen settings. We then randomly pick a starting N-gram, and show you a visualisation of the process of choosing the following characters till we have a new shiny business name!
                    </li>
                  </ol>

                  <p>
                    Make sure you experiment with the 'Left' and 'Right' values and see how it influences the final results. For further reading on character-based Markov Chains, we recommend taking a look at <a href="http://www.decontextualize.com/teaching/rwet/n-grams-and-markov-chains/">http://www.decontextualize.com/teaching/rwet/n-grams-and-markov-chains/</a>. Have fun!
                  </p>
                </div>
            </div>
        </div>
    </div>
</template>

<style>
.modal {
    text-shadow: none;
    color: #333
}

.modal-title {
    font-size: 18px;
}

.modal-body {
    font-size: 16px;
    text-align: left;
}

.modal a {
  color: inherit;
  font-weight: bold;
}
</style>
