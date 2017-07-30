<template>
    <div class="modal fade" id="helpModal" tabindex="-1" role="dialog" aria-labelledby="helpModalLabel">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h4 class="modal-title" id="helpModalLabel">About</h4>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
                </div>
                <div class="modal-body">
                    Intro text here
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
</style>