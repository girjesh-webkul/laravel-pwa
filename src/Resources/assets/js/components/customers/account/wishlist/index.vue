<template>
    <div class="content">
        <custom-header :title="$t('Wishlist')"></custom-header>

        <div class="panel" v-if="wishlist.length">
            <div class="panel-content">
                <div class="wishlist-list product-list product-grid-2">

                    <wishlist-card 
                        v-for="item in wishlist"
                        :key='item.uid'
                        :wishlistItem="item"
                        @onRemove="removeWishlistItem(item)"
                        @onMoveToCart="moveToCart(item)"
                    ></wishlist-card>

                </div>
            </div>
        </div>

        <empty-wishlist v-else></empty-wishlist>
    </div>
</template>

<script>
    import CustomHeader  from '../../../layouts/custom-header';
    import WishlistCard  from './card';
    import EmptyWishlist from './empty-wishlist';

    export default {
        name: 'wishlist',

        components: { CustomHeader, WishlistCard, EmptyWishlist },

        props: ['customer'],

        data () {
            return {
                wishlist: []
            }
        },

        mounted () {
            this.getWishList()
        },

        methods: {
            getWishList () {
                var this_this = this;

                EventBus.$emit('show-ajax-loader');

                this.$http.get('/api/pwa-wishlist', { params: { customer_id: this.customer.id, pagination: 0 ,token:true} })
                    .then(function(response) {
                        EventBus.$emit('hide-ajax-loader');

                        this_this.wishlist = response.data.data;
                    })
                    .catch(function (error) {});
            },

            removeWishlistItem (item) {
                var this_this = this;

                EventBus.$emit('show-ajax-loader');
                
                this.$http.delete('/api/wishlist/' + item.id)
                    .then(function(response) {
                        this_this.$toasted.show(response.data.message, { type: 'success' })

                        EventBus.$emit('hide-ajax-loader');

                        var index = this_this.wishlist.indexOf(item);

                        this_this.wishlist.splice(index, 1);
                    })
                    .catch(function (error) {});
            },

            moveToCart (item) {
                EventBus.$emit('show-ajax-loader');
                
                this.$http.get('/api/pwa/move-to-cart/' + item.id,{params : {token : true}})
                    .then(response => {
                        this.$toasted.show(response.data.message, { type: 'success' })

                        EventBus.$emit('hide-ajax-loader');

                        var index = this.wishlist.indexOf(item);

                        this.wishlist.splice(index, 1);
                        
                        EventBus.$emit('checkout.cart.changed', response.data.data);
                    })
                    .catch(error => {
                        if (error.response.data.data == -1) {
                            this.$router.push({ path: '/products/' + item.product.id })
                        } else if (error.response.data.message) {
                            this.$router.push({ path: '/products/' + item.product.id })

                            this.$toasted.show(error.response.data.message, { type: 'error' })
                        }
                    });
            }
        }
    }
</script>

<style scoped lang="scss">
    .content {
        position: absolute;
        bottom: 0;
        top: 0;
        width: 100%;
    }
</style>