<template>
  <div>
    <div class="grid grid-cols-2 svg-wrapper" ref="svg-wrapper">
      <div class="w-full h-screen p-2 flex-shrink-1">
        <div class="border">
          <div class="flex justify-items-stretch gap-2 p-4">
            <button
              @click="handleCategoryClick(category.id)"
              v-for="(category, index) in categories"
              :key="index"
              :class="`px-4 py-2 text-gray-800 text-sm font-medium rounded-md hover:bg-gray-300 flex items-center gap-2 border ${
                selectedCategory && selectedCategory.id === category.id ? 'bg-gray-300' : ''
              }`"
            >
              <span
                :class="{ 'bg-gray-200': category.color == null }"
                :style="{
                  backgroundColor: category.color || 'gray',
                  height: '10px',
                  width: '10px',
                  display: 'block',
                  borderRadius: '10px',
                }"
              ></span>
              {{ category.name }} | {{ category.curr }}
              {{ category.Ermäßigungen[0].price }}
            </button>

            <div v-if="isAdmin == 'true'" class="flex flex-row justify-end gap-2 flex-1">
              <label
                for="uploadSvg"
                class="px-4 py-2 text-gray-800 text-sm font-medium rounded-md hover:bg-gray-300 flex items-center gap-2 border cursor-pointer justify-self-end"
              >
                <input id="uploadSvg" type="file" @change="uploadFile" ref="file" class="hidden" />
                Upload SVG
              </label>
              <button
                @click="
                  () => {
                    onEdit = !onEdit;
                    if (!onEdit) {
                      selectedStatus = null;
                      selectedCategory = null;
                      applySeatColor();
                    }
                    if (onEdit) {
                      updateableSeats = {};
                    }
                  }
                "
                type="button"
                class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm py-1 px-2 text-center inline-flex items-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
              >
                Edit {{ onEdit ? 'On' : 'Off' }}
              </button>
            </div>
          </div>

          <div class="relative" v-if="src && src.length > 0">
            <div class="absolute flex flex-col top-0 right-5 z-10 ml-2 gap-2">
              <button
                @click="zoomIn"
                type="button"
                class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm py-1 px-2 text-center inline-flex items-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
              >
                +
              </button>

              <button
                @click="zoomOut"
                type="button"
                class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm py-1 px-2 text-center flex items-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
              >
                -
              </button>

              <button
                @click="resetZoom"
                type="button"
                class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm py-1 px-2 text-center flex items-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
              >
                R
              </button>

              <button
                @click="toggleZoomOrDrag"
                type="button"
                class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm py-1 px-2 text-center flex items-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
              >
                {{ showZoom ? 'Z' : 'D' }}
              </button>
            </div>

            <div ref="svg-preview">
              <div v-if="uploadedSvg != null" v-html="uploadedSvg" class="overflow-hidden"></div>
              <div v-else class="overflow-hidden">
                <inline-svg @loaded="svgLoaded($event)" :src="src" aria-label="My image"></inline-svg>
              </div>
            </div>
          </div>

          <div v-else>No source</div>
        </div>
      </div>

      <div class="w-full flex flex-row flex-shrink-1">
        <div class="w-1/3 px-2 overflow-y-scroll h-screen 5 relative p-2">
          <div class="m-1 border">
            <img src="map.svg" />
          </div>

          <h4 class="mt-10 mb-10">Blickrichtunk Buhne (Beispeil)</h4>

          <div class="mt-10">
            <img src="oreo.jpg" />
          </div>
          <div class="mt-10">
            <div
              v-for="(status, index) in statuses"
              class="flex gap-2 items-center"
              @mousedown="handleStatusClick(status.status_name)"
              :id="status.status_name"
              :key="index"
            >
              <span
                :class="selectedStatus == status.status_name ? 'border-green-500 font-bold' : ' border-black'"
                class="rounded-xl block bg-slate-500 rounded-xl border flex items-center justify-center"
                :style="{
                  height: '20px',
                  width: '20px',
                  borderStyle: status.border ? ` ${status.border} ` : ' solid',
                }"
              >
                <span
                  v-if="selectedStatus == status.status_name"
                  class="rounded-xl block bg-slate-500 rounded-xl border w-3 h-3 bg-green-500"
                ></span>
              </span>
              <label :for="status.status_name">{{ status.status_name }} </label>
            </div>
          </div>
        </div>
        <div class="w-2/3 h-screen 5 relative">
          <div class="shadow-md h-full flex flex-col">
            <!-- starthere -->

            <div class="rounded px-2 overflow-y-scroll h-screen grow">
              <div v-if="Object.keys(updateableSeats).length">
                <ul class="rounded overflow-hidden p-2">
                  <li
                    v-for="(seat, index) in updateableSeats"
                    :key="index"
                    class="shadow-md mb-2 flex flex-col px-4 py-2 hover:bg-sky-100 hover:text-sky-900 border last:border-none border-gray-200"
                  >
                    <div class="grid grid-cols-4 gap-4 mb-2">
                      <div class="flex flex-col border-r">
                        <span class="uppercase font-bold">Area</span>
                        <span> {{ seat.area }}</span>
                      </div>
                      <div class="flex flex-col border-r">
                        <span class="uppercase font-bold">Seat Number</span>
                        <span> {{ seat.seat }}</span>
                      </div>
                      <div class="flex flex-col">
                        <span class="uppercase font-bold">Row</span>
                        <span> {{ seat.row }}</span>
                      </div>
                      <div class="flex flex-col border-r">
                        <span class="uppercase font-bold">Category</span>
                        <span v-if="seat.category">
                          {{ seat.category.name }}
                        </span>
                        <span v-if="!seat.category" class="text-gray-300">Not selected </span>
                      </div>
                    </div>
                    <hr />
                    <div class="grid grid-cols-6 gap-4 mt-2">
                      <div class="flex flex-col col-span-2 border-r">
                        <span class="uppercase font-bold">Status</span>
                        <span v-if="seat.status"> {{ seat.status }}</span>
                        <span v-if="!seat.status" class="text-gray-300">Not selected </span>
                      </div>
                      <div class="flex flex-col col-span-3 border-r">
                        <span class="uppercase font-bold">Ermäßigungen</span>
                        <span v-if="!seat.category" class="text-gray-300">Not selected </span>
                        <select v-if="seat.category" @change="onChangeUpdateableSeatPrice($event, seat.id)">
                          <option
                            v-for="discount in seat.category.Ermäßigungen"
                            :key="discount.id"
                            :value="discount.id"
                            :title="seat.id"
                          >
                            {{ discount.name }} - {{ discount.price }}
                          </option>
                        </select>
                      </div>
                      <div class="col-span-1 flex items-center justify-center text-red-500">
                        <button @click="removeUpdateableSeat(seat.id)">X</button>
                      </div>
                    </div>
                  </li>
                </ul>
              </div>
              <div>
                <div v-if="Object.keys(selectedSeats).length">
                  <ul class="rounded overflow-hidden p-2">
                    <li
                      v-for="(seat, index) in selectedSeats"
                      :key="index"
                      class="relative shadow-md mb-2 flex flex-col px-4 py-2 hover:bg-sky-100 hover:text-sky-900 border last:border-none border-gray-200"
                    >
                      <div class="grid grid-cols-7 gap-4 mb-2">
                        <div class="flex flex-col col-span-3 border-r">
                          <span class="uppercase font-bold">Area</span>
                          <span> {{ seat.area }}</span>
                        </div>
                        <div class="flex flex-col col-span-2 border-r">
                          <span class="uppercase font-bold">Seat Number</span>
                          <span> {{ seat.seat }}</span>
                        </div>
                        <div class="flex flex-col col-span-2">
                          <span class="uppercase font-bold">Row</span>
                          <span> {{ seat.row }}</span>
                        </div>
                      </div>
                      <hr />
                      <div class="grid grid-cols-6 gap-4 mt-2">
                        <div class="flex flex-col col-span-2 border-r">
                          <span class="uppercase font-bold">Category</span>
                          <span> {{ seat.category.name }}</span>
                        </div>
                        <div class="flex flex-col col-span-3 border-r">
                          <span class="uppercase font-bold">Ermäßigungen</span>
                          <select @change="onChangeSelectedSeatPrice($event, seat.seat_id)">
                            <option
                              v-for="discount in seat.category.Ermäßigungen"
                              :key="discount.id"
                              :value="discount.id"
                              :title="seat.id"
                            >
                              {{ discount.name }} - {{ discount.price }}
                            </option>
                          </select>
                        </div>
                        <div class="col-span-1 flex items-center justify-center text-red-500">
                          <button @click="removeSelectedSeat(seat.seat_id)">X</button>
                        </div>
                      </div>
                    </li>
                  </ul>
                </div>
                <h1 class="text-center" v-else>No Seat Selected</h1>
              </div>
            </div>

            <!-- End Here -->

            <div class="bg-red-300 text-center">
              <button
                @click="
                  () => {
                    if (isAdmin == 'true') {
                      updateSeatData();
                    } else {
                    }
                  }
                "
                class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded m-2"
              >
                {{ isAdmin == 'true' ? 'Update Seat Data' : 'Tickets Kaufen' }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import InlineSvg from 'vue-inline-svg';
import DragSelect from 'dragselect';
import svgPanZoom from 'svg-pan-zoom';

export default {
  components: {
    InlineSvg,
  },
  name: 'OldView',

  mounted: function () {
    this.addCategoryDataToSeat();
    this.formatSeatColorData();

    //
  },
  props: ['src', 'isAdmin'],
  data: () => {
    return {
      onEdit: false,
      uploadedSvg: null,
      svgPan: null,
      dragSelect: null,
      showZoom: true,
      id: 'Not Selected',

      selectedCategory: null,
      selectedStatus: null,

      statuses: [
        { status_name: 'available', symbol_id: 1, border: null },
        { status_name: 'locked', symbol_id: 2, border: 'solid' },
        { status_name: 'reserved', symbol_id: 3, border: 'dotted' },
      ],
      categories: [
        {
          id: '1',
          name: 'Standard',
          curr: '$',
          color: 'cornflowerblue',
          Ermäßigungen: [
            {
              id: '5',
              name: 'Normalpreis',
              price: '130,90',
            },
            {
              id: '6',
              name: 'Special',
              price: '70.90',
            },
          ],
        },
        {
          id: '2',
          name: 'VIP',
          curr: '$',
          color: 'orange',
          Ermäßigungen: [
            {
              id: '7',
              name: 'Child',
              price: '100,90',
            },
            {
              id: '8',
              name: 'Kind',
              price: '55.90',
            },
          ],
        },
        {
          id: '3',
          name: 'VVIP',
          curr: '$',
          color: '#FF00FF',
          Ermäßigungen: [
            {
              id: '2',
              name: 'Adult',
              price: '150,90',
            },
            {
              id: '3',
              name: 'Boss',
              price: '79.90',
            },
          ],
        },
      ],
      seats: [
        {
          svg_id: '1',
          seat_id: 'e1_circle',
          area: 'Parterre',
          row: 'A',
          seat: 'A1',
          category_link: '1',
          discount_id: 5,
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'e16_circle',
          area: 'Parterre',
          row: 'A',
          seat: 'A2',
          category_link: '1',
          discount_id: 6,
          status: 'locked',
        },
        {
          svg_id: '1',
          seat_id: 'e2_circle',
          area: 'Parterre',
          row: 'A',
          seat: 'A3',
          category_link: '2',
          status: 'locked',
        },
        {
          svg_id: '1',
          seat_id: 'e8_circle',
          area: 'Parterre',
          row: 'A',
          seat: 'A4',
          category_link: '2',
          status: 'reserved',
        },
        {
          svg_id: '1',
          seat_id: 'e3_circle',
          area: 'Parterre',
          row: 'A',
          seat: 'A5',
          category_link: '2',
          status: 'reserved',
        },
        {
          svg_id: '1',
          seat_id: 'e10_circle',
          area: 'Parterre',
          row: 'B',
          seat: 'B1',
          category_link: '3',
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'e4_circle',
          area: 'Parterre',
          row: 'B',
          seat: 'B2',
          category_link: '3',
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'e11_circle',
          area: 'Parterre',
          row: 'B',
          seat: 'B3',
          category_link: '1',
          discount_id: 6,
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'e9_circle',
          area: 'Parterre',
          row: 'B',
          seat: 'B4',
          category_link: '2',
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'e12_circle',
          area: 'Parterre',
          row: 'B',
          seat: 'B5',
          category_link: '3',
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'e7_circle',
          area: 'Parterre',
          row: 'C',
          seat: 'C1',
          category_link: '3',
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'e21_circle',
          area: 'Parterre',
          row: 'C',
          seat: 'C2',
          category_link: '2',
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'e13_circle',
          area: 'Parterre',
          row: 'C',
          seat: 'C3',
          category_link: '1',
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'path889',
          area: 'Parterre',
          row: 'D',
          seat: 'D1',
          category_link: '2',
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'e14_circle',
          area: 'Parterre',
          row: 'D',
          seat: 'D2',
          category_link: '3',
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'e25_circle',
          area: 'Parterre',
          row: 'D',
          seat: 'D3',
          category_link: '2',
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'e15_circle',
          area: 'Parterre',
          row: 'D',
          seat: 'D4',
          category_link: '2',
          status: 'available',
        },
        {
          svg_id: '1',
          seat_id: 'e18_rectangle',
          area: 'Parterre',
          row: 'C',
          seat: 'C4',
          category_link: '1',
          status: 'available',
        },
      ],
      //seats: [],
      seatColors: {},
      selectedSeats: {},
      updateableSeats: {},
      controllSvgWidth: true,
      pos: { top: 0, left: 0, x: 0, y: 0 },
    };
  },

  methods: {
    svgLoaded() {
      this.applySeatColor();
      this.initPanZoom();
    },
    async uploadFile() {
      const file = this.$refs.file.files[0];
      this.uploadedSvg = await file.text();
      this.seats = [];
      this.selectedSeats = {};
      this.updateableSeats = {};
      this.onEdit = true;

      this.$nextTick(() => {
        if (this.showZoom) {
          this.destroyZoom();
          this.initPanZoom();
        } else {
          this.destroyDrag();
          this.initDragSelect();
        }
      });
    },
    //uploadedSvg
    toggleZoomOrDrag() {
      this.showZoom = !this.showZoom;
      if (this.showZoom) {
        this.enableZoom();
        this.destroyDrag();
      } else {
        this.initDragSelect();
        this.disableZoom();
      }
    },

    initPanZoom() {
      const ele = this.$refs['svg-preview'].querySelector('svg');
      this.svgPan = svgPanZoom(ele, {
        zoomEnabled: true,
        controlIconsEnabled: false,
        fit: true,
        center: true,
        dblClickZoomEnabled: false,
        // viewportSelector: document.getElementById('demo-tiger').querySelector('#g4') // this option will make library to misbehave. Viewport should have no transform attribute
      });
    },
    initDragSelect() {
      this.dragSelect = new DragSelect({
        selectables: document.querySelectorAll('g[label="clickable"] *'),
        immediateDrag: false,
        callback: (targets) => {
          console.log(targets.length);
          targets.forEach((target) => {
            this.manageSeatClick(target);
          });
          this.destroyDrag();
          this.initDragSelect();
        },
      });
    },
    applySeatColor() {
      const ref = this;

      this.seats.forEach((element) => {
        let seat = ref.$refs['svg-wrapper'].querySelector('#' + element.seat_id);
        console.log(seat);

        let color = this.seatColors[element.category_link];

        if (element.status === 'locked' || element.status === 'reserved') {
          const status = this.statuses.find((s) => s.status_name === element.status);
          seat.style.stroke = '#000000';
          seat.style.strokeWidth = '10px';
          if (status.border === 'dotted') {
            seat.style.strokeDasharray = '10, 10';
          }
        }
        if (color == null) {
          color = seat.style.fill;
        } else {
          seat.style.fill = color;
        }

        seat.setAttributeNS(null, 'default-color', color);
        //console.log()
      });
    },
    formatSeatColorData() {
      this.categories.map((element) => {
        this.seatColors[element.id] = element.color;
      });
    },
    handleClick(event) {
      const target = event.target;
      if (!this.showZoom) {
        return;
      }
      this.manageSeatClick(target);
    },

    manageSeatClick(target) {
      if (this.checkIfValidSeatObject(target)) {
        if (this.checkIfSeatIsNotChecked(target)) {
          const seatData = this.getSeatDataById(target.id);

          if (this.isAdmin == 'true' && this.onEdit) {
            this.constructSeatData(target);
          } else {
            if (
              seatData !== null &&
              // eslint-disable-next-line no-prototype-builtins
              seatData.hasOwnProperty('status') &&
              seatData.status === 'available'
            ) {
              this.setSeatAsChecked(target, seatData);
            }
          }
        } else {
          if (this.isAdmin == 'true' && this.onEdit) {
            this.removeConstructedData(target);
          } else {
            this.setSeatAsUnchecked(target);
          }
        }
      }
    },

    removeSelectedSeat(id) {
      let seat = this.$refs['svg-wrapper'].querySelector('#' + id);
      this.setSeatAsUnchecked(seat);
    },

    removeUpdateableSeat(id) {
      let seat = this.$refs['svg-wrapper'].querySelector('#' + id);
      this.removeConstructedData(seat);
    },

    handleStatusClick(e) {
      if (this.isAdmin != 'true' || !this.onEdit) {
        this.selectedStatus = '';
        return;
      }

      this.selectedStatus = e;
      //const seats = {};
      Object.keys(this.updateableSeats).forEach((value) => {
        this.updateableSeats[value].status = e;
      });

      return;
    },

    updateSeatData() {
      let newSeatData = this.seats;
      this.selectedStatus = '';

      Object.values(this.updateableSeats).forEach((seatData, index) => {
        let updateableIndex = null;

        let seat = this.$refs['svg-wrapper'].querySelector('#' + seatData.seat_id);

        let color = this.seatColors[seatData.category_link];
        if (color == null) {
          color = seat.getAttribute('default-color');
        }

        if (seat) {
          seat.style.fill = color;
          seat.setAttributeNS(null, 'checked', false);
          seat.setAttributeNS(null, 'default-color', color);
        }

        newSeatData.forEach((element, index) => {
          if (element.seat_id == seatData.seat_id) {
            updateableIndex = index;
            return;
          }
        });

        if (updateableIndex == null) {
          newSeatData.push(seatData);
        } else {
          newSeatData[index] = seatData;
        }
      });

      this.seats = newSeatData;
      this.applySeatColor();

      this.updateableSeats = {};
    },

    handleCategoryClick(id) {
      if (this.isAdmin != 'true' || !this.onEdit) {
        this.selectedCategory = null;
        return;
      }
      const seats = {};
      const category = this.getCategoryDataById(id);
      this.selectedCategory = category;

      Object.keys(this.updateableSeats).forEach((key) => {
        let value = this.updateableSeats[key];
        value.category = category;
        value.category_link = category.id;
        seats[key] = value;
      });
      this.updateableSeats = seats;

      console.log(this.updateableSeats);

      return;
    },

    constructSeatData(target, seatData = {}) {
      target.setAttributeNS(null, 'default-color', target.style.fill);
      target.setAttributeNS(null, 'checked', true);
      target.style.fill = 'green';
      //target.style.strokeWidth = '10px'

      const row = target.parentElement;
      const area = row.parentElement;
      //const root = area.parentElement;
      const id = target.id;
      //console.log(root.attributes)
      seatData = {
        ...seatData,
        seat_id: id,
        id: id,
        row: row.getAttribute('label'),
        area: area.getAttribute('label'),
      };

      if (this.selectedCategory) {
        seatData.category_link = this.selectedCategory.id;
        seatData.category = this.selectedCategory;
      }

      if (this.selectedStatus) {
        seatData.status = this.selectedStatus;
      }

      this.$set(this.updateableSeats, id, seatData);
    },
    removeConstructedData(target) {
      target.style.fill = target.getAttribute('default-color');
      target.setAttributeNS(null, 'checked', false);
      this.$delete(this.updateableSeats, target.id);
    },

    checkIfValidSeatObject(target) {
      return (
        typeof target === 'object' &&
        target.nodeType !== undefined &&
        (target.nodeName === 'circle' || target.nodeName === 'path' || target.nodeName === 'rect')
      );
    },
    checkIfSeatIsNotChecked(target) {
      const isChecked = target.getAttribute('checked');
      return isChecked == null || isChecked == undefined || isChecked == 'false';
    },

    setSeatAsChecked(target, seatData) {
      target.style.fill = 'green';
      target.setAttributeNS(null, 'checked', true);
      this.$set(this.selectedSeats, target.id, seatData);
    },

    setSeatAsUnchecked(target) {
      this.$delete(this.selectedSeats, target.id);
      target.style.fill = target.getAttribute('default-color');
      target.setAttributeNS(null, 'checked', false);
    },

    addCategoryDataToSeat() {
      this.seats = this.seats.map((element) => {
        element['category'] = this.getCategoryDataById(element.category_link);
        return element;
      });
    },

    onChangeSelectedSeatPrice(e, seatId) {
      this.$set(this.selectedSeats, seatId, {
        ...this.selectedSeats[seatId],
        ErmäßigungenId: e.target.value,
      });
    },

    onChangeUpdateableSeatPrice(e, seatId) {
      this.$set(this.updateableSeats, seatId, {
        ...this.updateableSeats[seatId],
        ErmäßigungenId: e.target.value,
      });
    },

    getSeatDataById(seatId) {
      let seatData;
      this.seats.forEach((element) => {
        if (seatId == element.seat_id) {
          seatData = element;
          return;
        }
      });
      return seatData;
    },
    getCategoryDataById(categoryId) {
      let categoryData;
      this.categories.forEach((element) => {
        if (categoryId == element.id) {
          categoryData = element;
          return;
        }
      });
      return categoryData;
    },

    zoomIn() {
      this.svgPan.zoomIn();
    },

    zoomOut() {
      this.svgPan.zoomOut();
    },

    resetZoom() {
      this.svgPan.resetZoom();
    },

    enableZoom() {
      this.svgPan.enablePan();
    },
    disableZoom() {
      this.svgPan.disablePan();
    },

    destroyZoom() {
      this.svgPan.destroy();
      //this.svgPan = null
    },

    destroyDrag() {
      this.dragSelect.stop();
      this.dragSelect = null;
    },
  },
};
</script>

<style scoped>
svg {
  max-width: 100%;
}
</style>
