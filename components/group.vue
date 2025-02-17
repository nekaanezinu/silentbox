<template>
    <section id="silentbox-group">
        <slot />
        <silentbox-overlay :overlay_class="overlay_class"/>
    </section>
</template>

<script>
    import overlay from './overlay.vue';

    export default {
        name: 'SilentboxGroup',
        props: ['overlay_class'],
        data() {
            return {
                overlayVisibility: false,
                embedUrl: '',
                items: {
                    total: 0,
                    position: 0,
                    list: []
                },
                autoplay: false,
                description: ''
            }
        },
        watch: {
            /**
             * Update total number of items when object changes.
             *
             * @param  {Object} current
             * @param  {Object} old
             * @return {void}
             */
            'items.list': function (current, old) {
                this.updateTotal(current);
            }
        },
        methods: {
            /**
             * Update count of total items in group.
             *
             * @param  {object} items
             * @return {void}
             */
            updateTotal(items) {
                this.items.total = this.items.list.length;
            },
            /**
             * Move to next item in a group.
             *
             * @return {void}
             */
            nextItem() {
                if (this.items.position !== (this.items.total - 1)) {
                    this.items.position++;
                } else {
                    this.items.position = 0;
                }

                this.embedUrl = this.items.list[this.items.position].src;

                this.autoplay = (this.items.list[this.items.position].autoplay !== undefined)
                    ? this.items.list[this.items.position].autoplay : false;

                this.description = (this.items.list[this.items.position].desc !== undefined)
                    ? this.items.list[this.items.position].desc : false;
            },
            /**
             * Move to previous item in a group.
             *
             * @return {void}
             */
            prevItem() {
                if (this.items.position !== 0) {
                    this.items.position--;
                } else {
                    this.items.position = this.items.total - 1;
                }

                this.embedUrl = this.items.list[this.items.position].src;

                this.autoplay = (this.items.list[this.items.position].autoplay !== undefined)
                    ? this.items.list[this.items.position] : false;

                this.description = (this.items.list[this.items.position].desc !== undefined)
                    ? this.items.list[this.items.position].desc : false;
            },
            /**
             * Set item that shuld be displayed.
             *
             * @return {void}
             */
            setOpened(item) {
                this.embedUrl = item.url;
                this.items.position = item.position;
                this.overlayVisibility = true;
                this.autoplay = item.autoplay;
                this.description = item.description;
            }
        },
        components: {
            'silentbox-overlay': overlay
        },
        mounted() {
            // Hide overlay when user click outside or on close button.
            this.$on('closeSilentboxOverlay', () => {
                this.overlayVisibility = false;
            });

            // Set first opened item when overlay opens.
            this.$on('openSilentboxOverlay', item => {
                this.setOpened(item);
            });

            // Update total number of available items in group.
            this.updateTotal(this.items);

            // Listen to key events.
            window.addEventListener('keyup', (event) => {
                // Escape: 27
                if (event.which === 27) {
                    this.overlayVisibility = false;
                }
                // Right arrow: 39
                if (event.which === 39) {
                    this.nextItem();
                }
                // Left arrow: 37
                if (event.which === 37) {
                    this.prevItem();
                }
            });
        }
    }
</script>
