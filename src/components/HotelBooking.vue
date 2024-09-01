<template>
  <div class="hotel-booking bg-light py-5">
    <div class="container">
      <div class="card shadow mb-5">
        <div class="card-header bg-primary text-white">
          <h2 class="h3 mb-0 fw-bold"><i class="bi bi-building me-2"></i>Book Your Dream Stay</h2>
          <p class="mb-0 small"><i class="bi bi-stars me-2"></i>Choose from our luxurious rooms and create unforgettable memories.</p>
        </div>
        <div class="card-body">
          <form @submit.prevent="submitBooking">
            <div class="row mb-4">
              <div class="col-md-4 mb-3 mb-md-0">
                <label for="checkIn" class="form-label"><i class="bi bi-calendar-check me-2"></i>Check-in Date</label>
                <input type="date" class="form-control" id="checkIn" v-model="booking.checkIn" required @change="validateDates">
              </div>
              <div class="col-md-4 mb-3 mb-md-0">
                <label for="checkOut" class="form-label"><i class="bi bi-calendar-x me-2"></i>Check-out Date</label>
                <input type="date" class="form-control" id="checkOut" v-model="booking.checkOut" required @change="validateDates">
              </div>
              <div class="col-md-4">
                <label for="guests" class="form-label"><i class="bi bi-people me-2"></i>Guests</label>
                <input type="number" class="form-control" id="guests" v-model="booking.guests" 
                       :min="1" :max="selectedRoom.maxGuests" required>
              </div>
            </div>

            <div v-if="dateMessage" class="alert" :class="dateMessageClass" role="alert">
              <i :class="dateMessageIcon"></i> {{ dateMessage }}
            </div>

            <div v-if="guestMessage" class="alert alert-info mb-4" role="alert">
              <i class="bi bi-info-circle me-2"></i>{{ guestMessage }}
            </div>

            <h3 class="h4 mb-3 fw-bold"><i class="bi bi-house-door me-2"></i>Select Your Room</h3>
            <div class="row">
              <div v-for="room in rooms" :key="room.id" class="col-md-4 mb-4">
                <div class="card h-100" :class="{ 'border-primary': booking.roomType === room.id }"
                  @click="selectRoom(room.id)" style="cursor: pointer;">
                  <img :src="room.image" class="card-img-top" :alt="room.name">
                  <div class="card-body">
                    <h4 class="card-title fw-bold">{{ room.name }}</h4>
                    <p class="card-text text-primary fw-bold"><i class="bi bi-currency-pound me-1"></i>{{ room.price }} / night</p>
                    <p class="card-text small"><i class="bi bi-person me-1"></i>Max guests: {{ room.maxGuests }}</p>
                    <p class="card-text small"><i class="bi bi-arrows-angle-expand me-1"></i>{{ room.size }} sq ft</p>
                  </div>
                </div>
              </div>
            </div>

            <div class="card bg-light mt-4 mb-4">
              <div class="card-body">
                <h4 class="card-title fw-bold"><i class="bi bi-receipt me-2"></i>Booking Summary</h4>
                <p class="card-text"><i class="bi bi-calendar3 me-2"></i>{{ calculateNights }} nights x £{{ selectedRoom.price }} per night</p>
                <p class="card-text h4 fw-bold"><i class="bi bi-cash me-2"></i>Total: £{{ selectedRoom.price * calculateNights }}</p>
              </div>
            </div>

            <div v-if="!isBookingValid" class="alert alert-warning mt-3" role="alert">
              <i class="bi bi-exclamation-triangle me-2"></i>The selected room can accommodate a maximum of {{ selectedRoom.maxGuests }} guests. Please adjust your selection.
            </div>

            <button type="submit" class="btn btn-primary btn-lg w-100 fw-bold" :disabled="!isBookingValid || !areDatesSelected">
              <i class="bi bi-check-circle me-2"></i>Book Now
            </button>
          </form>
        </div>
      </div>

      <div v-if="bookingConfirmed" class="alert alert-success mt-4" role="alert">
        <h4 class="alert-heading fw-bold"><i class="bi bi-check-circle-fill me-2"></i>Booking Confirmed!</h4>
        <p>Thank you for your reservation. We look forward to your stay!</p>
        <hr>
        <p class="mb-0">
          <strong><i class="bi bi-calendar-check me-2"></i>Check-in:</strong> {{ booking.checkIn }}<br>
          <strong><i class="bi bi-calendar-x me-2"></i>Check-out:</strong> {{ booking.checkOut }}<br>
          <strong><i class="bi bi-people me-2"></i>Guests:</strong> {{ booking.guests }}<br>
          <strong><i class="bi bi-house-door me-2"></i>Room:</strong> {{ selectedRoom.name }}
        </p>
      </div>

      <!-- Booking List Section -->
      <div class="card shadow mt-5">
        <div class="card-header bg-secondary text-white">
          <h3 class="h4 mb-0 fw-bold"><i class="bi bi-list-ul me-2"></i>Current Bookings</h3>
        </div>
        <div class="card-body">
          <div v-if="bookings.length === 0" class="alert alert-info">
            <i class="bi bi-info-circle me-2"></i>No bookings have been made yet.
          </div>
          <div v-else class="list-group">
            <div v-for="(booking, index) in bookings" :key="index" class="list-group-item">
              <h5 class="mb-1"><i class="bi bi-bookmark me-2"></i>Booking #{{ index + 1 }}</h5>
              <p class="mb-1"><strong><i class="bi bi-house-door me-2"></i>Room:</strong> {{ getRoomName(booking.roomType) }}</p>
              <p class="mb-1"><strong><i class="bi bi-calendar-check me-2"></i>Check-in:</strong> {{ booking.checkIn }}</p>
              <p class="mb-1"><strong><i class="bi bi-calendar-x me-2"></i>Check-out:</strong> {{ booking.checkOut }}</p>
              <p class="mb-1"><strong><i class="bi bi-people me-2"></i>Guests:</strong> {{ booking.guests }}</p>
              <p class="mb-0"><strong><i class="bi bi-cash me-2"></i>Total:</strong> £{{ calculateTotal(booking) }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'HotelBooking',
  data() {
    return {
      booking: {
        checkIn: '',
        checkOut: '',
        guests: 1,
        roomType: '1'
      },
      rooms: [
        { id: '1', name: 'Standard Single', price: 100, maxGuests: 1, size: 200, image: 'https://www.thespruce.com/thmb/Ej4ll-zmgvdy69_9lSiT5fQDifA=/750x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/bedroom-nightstand-essentials-350510-hero-625142f9f06c4d789a245a960d2625fb.jpg' },
        { id: '2', name: 'Standard Double', price: 150, maxGuests: 2, size: 250, image: 'https://www.thespruce.com/thmb/LYx2uVFAedcNtupcWxw9yGVkA1U=/750x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/Ucpage_GettyImages-80f5b7a685ce4204a39528ff20ef52b2.jpg' },
        { id: '3', name: 'Deluxe Single', price: 200, maxGuests: 1, size: 300, image: 'https://www.thespruce.com/thmb/Afg3IVBq0tV-7DHBME5woSNCZxQ=/750x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/put-together-a-perfect-guest-room-1976987-hero-223e3e8f697e4b13b62ad4fe898d492d.jpg' },
        { id: '4', name: 'Deluxe Double', price: 250, maxGuests: 2, size: 350, image: 'https://www.marthastewart.com/thmb/JlKSNX1MKC3eHUd-sbHTh79VCJQ=/750x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/pacific-northwest-home-tour-great-room-0820-14d61b428237459b9e996c769ae92dd0.jpg' },
        { id: '5', name: 'Junior Suite', price: 300, maxGuests: 2, size: 400, image: 'https://www.marthastewart.com/thmb/z_IjEUsXycGCCRQR-paAgvgLVuo=/750x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/modern-living-rooms-wb-1-bc45b0dc70e541f0ba40364ae6bd8421.jpg' },
        { id: '6', name: 'Executive Suite', price: 400, maxGuests: 2, size: 500, image: 'https://www.thespruce.com/thmb/wXt2EtpthEh-lIK6j3NZR67cqY8=/750x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/aesthetic-room-ideas-5195645-hero-7d51313f2c8f4ed6b338513ae284b113.jpg' },
        { id: '7', name: 'Family Room', price: 350, maxGuests: 4, size: 450, image: 'https://www.thespruce.com/thmb/vt93nbczu46qXLdRflZ7G7dJRyQ=/750x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/316825922_1266325077246930_4112521625154327780_n-0d56d97e634346fbba466f77ab111214.jpg' },
      ],
      bookings: [],
      bookingConfirmed: false,
      dateMessage: 'Please select your check-in and check-out dates.',
      dateMessageClass: 'alert-info'
    }
  },
  computed: {
    selectedRoom() {
      return this.rooms.find(room => room.id === this.booking.roomType) || this.rooms[0];
    },
    calculateNights() {
      if (!this.booking.checkIn || !this.booking.checkOut) return 0;
      const checkInDate = new Date(this.booking.checkIn);
      const checkOutDate = new Date(this.booking.checkOut);
      const timeDifference = checkOutDate.getTime() - checkInDate.getTime();
      return Math.max(Math.ceil(timeDifference / (1000 * 3600 * 24)), 1);
    },
    isBookingValid() {
      if (!this.selectedRoom) return false;
      return this.booking.guests <= this.selectedRoom.maxGuests && this.areDatesValid;
    },
    guestMessage() {
      if (!this.areDatesSelected) return '';
      if (this.booking.guests > this.selectedRoom.maxGuests) {
        return `You've selected ${this.booking.guests} guests, but the ${this.selectedRoom.name} can only accommodate ${this.selectedRoom.maxGuests}. Please select a different room or reduce the number of guests.`;
      } else if (this.booking.guests === this.selectedRoom.maxGuests) {
        return `Perfect! The ${this.selectedRoom.name} is ideal for your group of ${this.booking.guests}.`;
      } else {
        return `You've selected ${this.booking.guests} guest(s) for the ${this.selectedRoom.name}. This room can accommodate up to ${this.selectedRoom.maxGuests} guests.`;
      }
    },
    areDatesSelected() {
      return this.booking.checkIn && this.booking.checkOut;
    },
    areDatesValid() {
      if (!this.areDatesSelected) return false;
      const checkInDate = new Date(this.booking.checkIn);
      const checkOutDate = new Date(this.booking.checkOut);
      return checkOutDate > checkInDate;
    },
    dateMessageIcon() {
      return this.areDatesValid ? 'bi bi-calendar-check me-2' : 'bi bi-calendar-x me-2';
    }
  },
  methods: {
    selectRoom(roomId) {
      this.booking.roomType = roomId;
      if (this.booking.guests > this.selectedRoom.maxGuests) {
        this.booking.guests = this.selectedRoom.maxGuests;
      }
    },
    submitBooking() {
      if (this.isBookingValid && this.areDatesSelected) {
        this.bookings.push({ ...this.booking });
        console.log('Booking submitted:', this.booking);
        this.bookingConfirmed = true;
        setTimeout(() => {
          this.bookingConfirmed = false;
          this.booking = {
            checkIn: '',
            checkOut: '',
            guests: 1,
            roomType: '1'
          };
          this.dateMessage = 'Please select your check-in and check-out dates.';
          this.dateMessageClass = 'alert-info';
        }, 5000);
      }
    },
    validateDates() {
      if (!this.areDatesSelected) {
        this.dateMessage = 'Please select both check-in and check-out dates.';
        this.dateMessageClass = 'alert-info';
      } else if (!this.areDatesValid) {
        this.dateMessage = 'Check-out date must be after check-in date.';
        this.dateMessageClass = 'alert-danger';
      } else {
        this.dateMessage = `You've selected a ${this.calculateNights}-night stay.`;
        this.dateMessageClass = 'alert-success';
      }
    },

    getRoomName(roomId) {
      const room = this.rooms.find(room => room.id === roomId);
      return room ? room.name : 'Unknown Room';
    },
    calculateTotal(booking) {
      const room = this.rooms.find(room => room.id === booking.roomType);
      if (!room) return 0;
      
      const checkInDate = new Date(booking.checkIn);
      const checkOutDate = new Date(booking.checkOut);
      const nights = Math.max(Math.ceil((checkOutDate - checkInDate) / (1000 * 60 * 60 * 24)), 1);
      
      return room.price * nights;
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;700&display=swap');

body {
  font-family: 'Plus Jakarta Sans', sans-serif;
}

.hotel-booking {
  font-family: 'Plus Jakarta Sans', sans-serif;
}

.card {
  transition: all 0.3s ease;
  overflow: hidden;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.card-img-top {
  height: 200px;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.card:hover .card-img-top {
  transform: scale(1.05);
}

.fw-bold {
  font-weight: 700 !important;
}

.btn {
  font-weight: 500;
}

/* New styles for icons */
.bi {
  vertical-align: -0.125em;
}

.alert .bi {
  font-size: 1.2em;
}

.card-title .bi,
.h4 .bi {
  font-size: 1.1em;
}
</style>