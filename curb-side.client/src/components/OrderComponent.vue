<template>
  <div :class="'row lines mt-3 pb-2'" v-if="state.showDate && !state.activeArchive">
    <div class="col">
      <p>{{ getDate() }}</p>
    </div>
  </div>
  <div class="row lines p-3" :class="state.color">
    <div class="col-6 p-0">
      <div class="d-flex">
        <p class="mr-1">
          <b>Order Recieved:</b>
        </p>
        <p><b>{{ time() }}</b></p>
      </div>

      <order-item-component v-for="item in orderProp.contents" :key="item._id" :item-prop="item" />
      <div class="container-fluid">
        <div class="row">
          <div class="col-9">
            <p>total charge:</p>
          </div>
          <div class="col-3">
            <p>${{ orderProp.subTotal }}</p>
          </div>
        </div>
      </div>
    </div>
    <div class="col-6 d-flex align-items-center flex-column pr-0" v-if="state.activeArchive">
      <div class="container-fluid">
        <div class="row">
          <div class="col-7 d-flex align-items-end">
            <p class="mr-2">
              {{ orderProp.creator.name }}
            </p>
          </div>
          <div class="offset-1 col-3">
            <i class="fa fa-comment fa-2x pointer mb-1 ml-1 mr-3 align-self-center" aria-hidden="true" data-toggle="modal" @click="formatModal" :data-target="'#chat'+ orderProp._id"></i>
            <div :id="'id-'+orderProp._id" class="ping" :class="{'vis': orderProp.unreads}"></div>
          </div>
        </div>
        <div class="row">
          <div class="col-12">
            <p class="green-text" v-if="orderProp.here">
              The customer is waiting!
            </p>
            <p class="red-text" v-else>
              The customer is not here yet!
            </p>
          </div>
        </div>
        <div class="row">
          <div class="col-12 fs-6">
            <button class="btn-outline-success btn btn-sm text-sm min-width" data-toggle="modal" :data-target="'#id' + orderProp._id">
              Order Completed
            </button>
          </div>
          <div class="col-12">
            <button class="btn-outline-secondary btn btn-sm text-sm min-width" data-toggle="modal" :data-target="'#canid' + orderProp._id">
              Cancel Order
            </button>
          </div>
        </div>
      </div>
    </div>
    <div class="col-6" v-else>
      <p class="text-primary" v-if="orderProp.status === 'completed'">
        This order was fulfilled
      </p>
      <p class="text-secondary" v-else-if="orderProp.status==='cancelledByBusiness'">
        This order was cancelled by business
      </p>
      <p class="text-secondary" v-else-if="orderProp.status==='cancelledByCustomer'">
        This order was cancelled by customer
      </p>
    </div>
  </div>
  <complete-order-modal :complete-prop="orderProp._id" />
  <cancel-order-modal :cancel-prop="orderProp._id" />
  <!-- Modal -->
  <div class="modal fade bg-light"
       data-backdrop="static"
       :id="'chat'+ orderProp._id"
       tabindex="-1"
       aria-labelledby="exampleModalLabel"
       aria-hidden="true"
  >
    <div class="modal-dialog  modal-fullscreen-sm-down">
      <div class="modal-content">
        <div class="d-flex justify-content-end">
          <i class="fa fa-times-circle-o" @click="closeModal(orderProp._id)" aria-hidden="true"></i>
        </div>
        <div class="chat-div modal-body col" v-if="orderProp.chat">
          <chat-component :chat-prop="orderProp.chat" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { computed, onMounted, reactive } from 'vue'
import { chatService } from '../services/ChatService'
import { logger } from '../utils/Logger'
import { AppState } from '../AppState'
import $ from 'jquery'
export default {
  name: 'OrderComponent',
  props: {

    orderProp: {
      type: Object,
      required: true
    },
    archived: {
      type: Boolean
    }
  },
  setup(props) {
    const state = reactive({
      chat: computed(() => AppState.chat),
      message: '',
      orderDate: computed(() => AppState.date),
      showDate: false,
      activeArchive: true,
      color: ''
    })
    onMounted(() => {
      const index = AppState.orders.findIndex(o => o._id === props.orderProp._id)
      if (index % 2 === 0) {
        state.color = 'white'
      } else {
        state.color = 'green'
      }

      if (props.orderProp.date === state.orderDate) {
        state.showDate = false
      } else {
        state.showDate = true
      }
      AppState.date = props.orderProp.date
      if (props.archived) {
        state.activeArchive = false
      }
    })
    return {
      state,
      formatModal() {
        const order = AppState.orders.find(o => o._id === props.orderProp._id)
        // console.log(order)
        order.unreads = false
        setTimeout(() => {
          // console.log('#id-' + props.orderProp.chat._id)
          const height = $('#id-' + props.orderProp.chat._id)[0].scrollHeight
          // console.log(height)
          $('#id-' + props.orderProp.chat._id).animate({ scrollTop: height }, 200, 'swing')
        }, 300)
      },
      time() {
        const date = new Date(props.orderProp.createdAt)
        let hours = date.getHours()
        const minutes = date.getMinutes()
        if (hours > 12) {
          hours -= 12
        }
        return hours + ':' + minutes
      },

      // async getChat() {
      //   try {
      //     socketService.emit('join:room', props.orderProp._id)
      //     await chatService.getChats(props.orderProp._id)
      //   } catch (error) {
      //     logger.error(error)
      //   }
      // },
      closeModal(id) {
        $('#chat' + id).modal('hide')
        // socketService.emit('leave:room', id)
        AppState.chat = {}
      },
      async createMessage() {
        try {
          await chatService.createMessage(state.message)
          state.message = ''
        } catch (error) {
          logger.error(error)
        }
      },
      getDate() {
        const date = new Date(props.orderProp.createdAt)
        let month = date.getMonth()
        const year = date.getFullYear()
        const day = date.getDate()
        month += 1
        switch (month) {
          case 1:
            month = 'January'
            break
          case 2:
            month = 'Febuary'
            break
          case 3:
            month = 'March'
            break
          case 4:
            month = 'April'
            break
          case 5:
            month = 'May'
            break
          case 6:
            month = 'June'
            break
          case 7:
            month = 'July'
            break
          case 8:
            month = 'August'
            break
          case 9:
            month = 'September'
            break
          case 10:
            month = 'October'
            break
          case 11:
            month = 'November'
            break
          case 12:
            month = 'December'
            break
        }
        return month + '-' + day + '-' + year
      }
    }
  }
}
</script>

<style scoped lang='scss'>
@import '../assets/scss/_variables.scss';
p{
  margin-bottom: 0;
}
.green-text{
  color: var(--primary)
}
.red-text{
  color: var(--secondary)
}
.text-sm{
  font-size: .8rem !important;
}
.min-width{
  width: 100%;
}
.chat-div {
  height: 80vh;
  overflow: hidden;
  display: flex !important;
}
.ping {
  border-radius: 50%;
  background-color: $danger ;
  height: 14px;
  width: 14px;
  border: 1px solid white;
  transform: translateX(20px) translateY(-35px);
  visibility: hidden;
}
.vis {
  visibility:visible;
}
</style>
