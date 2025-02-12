<template>
  <div>
    <h2></h2>
    <div class="textarea-container">
      <div class="menu-bar">
        <div style="flex-grow: 1; text-align: left;">JSON Editor for Digilocker CSV</div>
        <button class="format-btn" @click="formatJSON"><i class="fas fa-align-left"></i></button>
        <button class="expand-btn" @click="toggleTextareaSize"><i class="fas fa-expand"></i></button>
      </div>
      <textarea v-model="jsonInput" placeholder="Paste your JSON here"></textarea>
    </div>
    <div class="button-group">
      <button @click="loadJSON"><i class="fas fa-upload"></i> Load JSON</button>
      <label for="csvInput" class="button-group">
        <i class="fas fa-file-csv"></i> Load CSV
        <input type="file" id="csvInput" accept=".csv" @change="loadCSV" style="display: none;">
      </label>
      <button @click="sortTable"><i class="fas fa-sort"></i> Sort by Order</button>
      <button @click="correctOrder"><i class="fas fa-check"></i> Correct Order</button>
      <button @click="resetJSON"><i class="fas fa-redo"></i> Reset</button>
    </div>

    <table id="jsonTable" v-show="tableVisible">
      <thead id="tableHead" v-show="tableVisible">
        <tr>
          <th>Order</th>
          <th>Head Name</th>
          <th>Code</th>
          <th>Display Name</th>
          <th>Is Subject</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(tag, index) in csvTags" :key="index">
          <td><input type="text" v-model="tag.order" @input="checkForDuplicates"></td>
          <td><input type="text" v-model="tag.headName" @input="checkForDuplicates"></td>
          <td><input type="text" v-model="tag.code"></td>
          <td><input type="text" v-model="tag.displayName" @input="checkForDuplicates"></td>
          <td><input type="checkbox" v-model="tag.isSubject"></td>
        </tr>
      </tbody>
    </table>

    <div class="button-group">
      <button @click="addRow"><i class="fas fa-plus"></i> Add Row</button>
      <button @click="exportJSON"><i class="fas fa-download"></i> Export JSON</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      jsonInput: '',
      csvTags: [],
      tableVisible: false,
    };
  },
  methods: {
    loadJSON() {
      let jsonObject;

      try {
        jsonObject = JSON.parse(this.jsonInput);
      } catch (error) {
        alert('Invalid JSON');
        return;
      }

      this.populateTable(jsonObject.csvTags);
    },
    populateTable(csvTags) {
      this.csvTags = csvTags;
      this.tableVisible = true;
      this.checkForDuplicates();
    },
    exportJSON() {
      const jsonObject = { csvTags: this.csvTags };
      const jsonString = JSON.stringify(jsonObject, null, 2);
      const blob = new Blob([jsonString], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'modified_json.json';
      a.click();
      URL.revokeObjectURL(url);
    },
    loadCSV(event) {
      const input = event.target;
      if (input.files.length === 0) {
        alert('Please select a CSV file');
        return;
      }

      const file = input.files[0];
      const reader = new FileReader();
      reader.onload = (e) => {
        const text = e.target.result;
        const csvTags = this.parseCSV(text);

        this.populateTable(csvTags);
        this.jsonInput = JSON.stringify({ csvTags }, null, 2);
      };
      reader.readAsText(file);
    },
    parseCSV(text) {
      const lines = text.split('\n');
      const headersString = lines[0].replace(/\t/g, ',');
      const headers = headersString.split(',').map(header => header.trim());
      const csvTags = [];
      const codeMap = {
        "ORG_NAME": "collegeName",
        "ACADEMIC_COURSE_ID": "degreeName",
        "COURSE_NAME": "degreeName",
        "ADMISSION": "admissionYear",
        "STREAM": "batchNameWithoutYear",
        "SESSION": "sessionName",
        "REGN_NO": "registerNo",
        "RROLL": "registerNo",
        "CNAME": "studentName",
        "GENDER": "genderFirstLetter",
        "DOB": "dob",
        "FNAME": "fatherName",
        "MNAME": "motherName",
        "PHOTO": "studentImageRegNo",
        "MRKS_REC_STATUS": "omcStatus",
        "RESULT": "class",
        "YEAR": "examYear",
        "MONTH": "examMonthNameFull",
        "PERCENT": "semesterPercentage",
        "CERT_NO": "slNo",
        "SEM": "academicTermNameInRoman",
        "TOT": "grandTotalMark",
        "TOT_MRKS": "semesterTotalMarksObtained",
        "TOT_CREDIT": "earnedCreditWithExcludeTotalCondition",
        "TOT_GRADE_POINTS": "totalGradePoint",
        "SGPA": "sgpa",
        "TOT_GRADE": "semesterGrandTotalGrade"
      };

      headers.forEach((header, index) => {
        if ((header.startsWith("SUB1") && !header.startsWith("SUB10")) || !header.startsWith("SUB")) {
          const isSubject = header.startsWith("SUB");
          const code = isSubject ? header.slice(0, -3).toLowerCase() : (codeMap[header] || header.toLowerCase());
          const order = index + 1;
          let displayName = header;
          let headName = header;
          if (isSubject) {
            headName = header.slice(4);
            displayName = header.slice(0, 3) + header.slice(4);
          }

          csvTags.push({
            code,
            order,
            headName,
            isSubject,
            displayName
          });
        }
      });

      return csvTags;
    },
    sortTable() {
      this.csvTags.sort((a, b) => a.order - b.order);
      this.checkForDuplicates();
    },
    correctOrder() {
      this.csvTags.forEach((tag, index) => {
        tag.order = index + 1;
      });
      this.checkForDuplicates();
    },
    addRow() {
      this.csvTags.push({
        order: this.csvTags.length + 1,
        headName: '',
        code: '',
        displayName: '',
        isSubject: false,
      });
      this.tableVisible = true;
      this.checkForDuplicates();
    },
    checkForDuplicates() {
      const orderValues = this.csvTags.map(tag => tag.order);
      const headNameValues = this.csvTags.map(tag => tag.headName);
      const displayNameValues = this.csvTags.map(tag => tag.displayName);
      const isSubjectValues = this.csvTags.map(tag => tag.isSubject);

      // Reset all inputs
      this.csvTags.forEach(tag => {
        tag.orderError = false;
        tag.headNameError = false;
        tag.displayNameError = false;
      });

      // Find duplicates in orders
      const duplicateOrders = orderValues.filter((value, index, self) => self.indexOf(value) !== index && value !== "");

      // Highlight duplicate orders
      duplicateOrders.forEach(duplicate => {
        this.csvTags.forEach(tag => {
          if (tag.order === duplicate) {
            tag.orderError = true;
          }
        });
      });

      // Find duplicates in headName and isSubject combination
      const uniqueCombinations = new Set();
      const duplicateCombinations = new Set();
      this.csvTags.forEach((tag, index) => {
        const key = `${tag.headName}-${tag.isSubject}`;

        if (uniqueCombinations.has(key)) {
          duplicateCombinations.add(key);
        } else {
          uniqueCombinations.add(key);
        }
      });

      // Highlight duplicate headName and isSubject combinations
      duplicateCombinations.forEach(duplicate => {
        this.csvTags.forEach(tag => {
          const key = `${tag.headName}-${tag.isSubject}`;
          if (key === duplicate) {
            tag.headNameError = true;
          }
        });
      });

      // Find duplicates in displayName
      const duplicateDisplayNames = displayNameValues.filter((value, index, self) => self.indexOf(value) !== index && value !== "");

      // Highlight duplicate displayNames
      duplicateDisplayNames.forEach(duplicate => {
        this.csvTags.forEach(tag => {
          if (tag.displayName === duplicate) {
            tag.displayNameError = true;
          }
        });
      });
    },
    resetJSON() {
      this.jsonInput = '';
      this.csvTags = [];
      this.tableVisible = false;
    },
    toggleTextareaSize() {
      const textarea = this.$refs.jsonInput;
      const expandBtn = this.$refs.expandBtn;
      if (textarea.style.height === '400px') {
        textarea.style.height = '150px';
        expandBtn.className = 'fas fa-expand';
      } else {
        textarea.style.height = '400px';
        expandBtn.className = 'fas fa-compress';
      }
    },
    formatJSON() {
      try {
        const jsonObject = JSON.parse(this.jsonInput);
        this.jsonInput = JSON.stringify(jsonObject, null, 2);
      } catch (error) {
        alert('Invalid JSON');
      }
    },
  },
};
</script>

<style scoped>
@import '../assets/styles.css';
</style>