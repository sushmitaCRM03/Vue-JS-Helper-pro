<template>
  <div
    class="modal fade"
    :class="['modal-mask', 'show', 'd-block']"
    tabindex="-1"
    aria-labelledby="exampleModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog h-100 mw-80" style="max-width: 90%">
      <div class="modal-content modal-dialog-scrollable">
        <div class="row justify-content-center align-items-center m-0">
          <!-- Candidate Detail (Left Side) -->
          <div id="leftColumn" class="col-9 mb-4">
            <UIPageHeaderResponsive>
              <template #preHeaderActions>
                <div class="me-2"></div>
                <div class="me-2">
                  <img
                    class="image avatar-text-45"
                    v-if="
                      candidateDetail?.profilepic ||
                        candidateDetail?.profilepicUrl
                    "
                    :src="
                      candidateDetail?.profilepicUrl?.split(',')[0] ||
                        candidateDetail?.profilepic?.split(',')[0]
                    "
                    alt=""
                  />
                  <div
                    v-else
                    class="avatar-text-45 bg-candidate color-candidate"
                  >
                    <span>
                      {{ getAcronym(candidateDetail.candidatename) }}
                    </span>
                  </div>
                </div>
              </template>
              <template #pageHeader>
                <div class="row">
                  <span class="fwt-bold fs-16">{{
                    candidateDetail.candidatename
                  }}</span>
                  <div
                    class="fs-12 fwt-bold d-flex align-items-center justify-content-start mt-1"
                  >
                    <div
                      v-if="
                        checkVisiblity('emailid') && candidateDetail['emailid']
                      "
                      class="me-4"
                    >
                      <img
                        src="@/assets/images/mail.svg"
                        alt="No Records Found"
                      />
                      <span class="ms-1 fs-13 text-muted">
                        {{ getCandidateDetail(candidateDetail.emailid) }}
                      </span>
                    </div>
                    <div
                      v-if="
                        checkVisiblity('contactnumber') &&
                          candidateDetail['contactnumber']
                      "
                    >
                      <img
                        src="@/assets/images/phone.svg"
                        alt="No Records Found"
                      />
                      <span class="ms-1 text-muted">
                        {{ getCandidateDetail(candidateDetail.contactnumber) }}
                      </span>
                    </div>
                  </div>
                </div>
              </template>
              <template #pageActionsRight>
                <div class="d-flex justify-content-end">
                  <div class="text-end me-2 d-grid" v-if="visibilityOfStage">
                    <span class="text-muted fwt-bold mb-2"
                      >Agency Hiring Stage
                    </span>
                    <span
                      v-if="checkVisiblity('candidatestatus')"
                      class="badge rounded-pill stage-badge p-2 fs-12 fwt-bold"
                      >{{
                        jobData.candidateHiringStages.externalPipeline
                      }}</span
                    >
                  </div>
                  <div
                    class="vr me-2"
                    v-if="visibilityOfStage && isPortalViewDisabled"
                  ></div>
                  <div v-if="isPortalViewDisabled">
                    <div
                      class="d-grid"
                      v-if="jobData.jobCandidatesById.addedToInternalPipeline"
                    >
                      <span class="text-muted fwt-bold mb-2"
                        >Internal Hiring Stage</span
                      >
                      <span
                        class="badge rounded-pill stage-badge p-2 fs-12 fwt-bold"
                      >
                        {{ jobData.candidateHiringStages.internalPipeline }}
                      </span>
                    </div>
                    <div class="mt-2" v-else>
                      <button
                        v-if="
                          ability.can(
                            abilities.UPDATE,
                            accessControlModules.CANDIDATES
                          )
                        "
                        id="sTest_mapToJobBtn"
                        class="btn btn-icon-text border fwt-bold"
                        @click="addToInternalPipeline"
                      >
                        <em class="bx bx-plus me-1"></em>
                        Add to Internal Pipeline
                      </button>
                    </div>
                  </div>
                </div>
              </template>
            </UIPageHeaderResponsive>
            <div class="card mt-4">
              <div class="card-body p-3">
                <h5 class="card-title pt-1">Candidate Details</h5>
                <div class="row d-flex align-items-center p-3">
                  <div
                    v-for="(column, columnKey) in allVisibleFields"
                    :key="columnKey"
                    class="col-6 margin-check"
                  >
                    <div class="row">
                      <div class="col-6 fwt-bold">
                        {{ column.label }}
                      </div>
                      <div v-if="isCustomField(columnKey)" class="col-6">
                        <div
                          v-if="
                            candidateDetail[columnKey].column_type == 'file'
                          "
                        >
                          <button
                            class="btn btn-icon justify-content-start me-2"
                            v-if="
                              candidateDetail[columnKey].column_value != null
                            "
                            @click="
                              viewFileModal(
                                candidateDetail[columnKey].column_value
                              )
                            "
                          >
                            <em
                              class="bx bxs-file bx-sm"
                              style="color:#2517d8"
                            ></em>
                          </button>
                          <button
                            class="btn btn-icon justify-content-start me-2"
                            v-else
                          >
                            <em class="bx bx-file-blank bx-sm"></em>
                          </button>
                        </div>
                        <div
                          v-else-if="
                            candidateDetail[columnKey].column_type == 'longtext'
                          "
                        >
                          <span>{{
                            candidateDetail[columnKey].column_value
                              ? $truncate(
                                  candidateDetail[columnKey].column_value,
                                  { length: 15 }
                                )
                              : "Not Available"
                          }}</span>
                          <a
                            v-if="candidateDetail[columnKey].column_value"
                            href="#"
                            class="ms-1 stretched-link position-relative"
                            @click="
                              openLongTextModal(candidateDetail[columnKey])
                            "
                            >View More</a
                          >
                        </div>
                        <div v-else>
                          <span
                            v-if="
                              candidateDetail[columnKey].column_type == 'date'
                            "
                            >{{
                              candidateDetail[columnKey].column_value
                                ? getDate(
                                    candidateDetail[columnKey].column_value
                                  )
                                : "Not Available"
                            }}</span
                          >
                          <span
                            v-else-if="
                              candidateDetail[columnKey].column_type ==
                                'checkbox'
                            "
                            >{{
                              candidateDetail[columnKey].column_value
                                ? candidateDetail[columnKey].column_value == 1
                                  ? "Yes"
                                  : "No"
                                : "Not Available"
                            }}</span
                          >
                          <span
                            v-else
                            data-bs-toggle="tooltip"
                            data-bs-placement="top"
                            :data-bs-title="
                              candidateDetail[columnKey].column_value
                            "
                            data-bs-custom-class="custom-tooltip"
                            >{{
                              candidateDetail[columnKey].column_value
                                ? $truncate(
                                    candidateDetail[columnKey].column_value,
                                    { length: 15 }
                                  )
                                : "Not Available"
                            }}
                          </span>
                        </div>
                      </div>

                      <span
                        v-else-if="fileTypes.includes(columnKey)"
                        class="col-6 d-flex align-items-center"
                      >
                        <div v-if="columnKey == 'summary'">
                          <button
                            class="btn btn-icon justify-content-start me-2"
                            v-if="candidateDetail[columnKey]"
                            @click="viewSummaryModal"
                          >
                            <img
                              src="@/assets/images/summary-icon-filled.svg"
                              alt=""
                              width="20"
                              height="20"
                            />
                          </button>
                          <button
                            class="btn btn-icon justify-content-start me-2"
                            v-else
                          >
                            <img
                              src="@/assets/images/summary-icon-blank.svg"
                              alt=""
                              width="20"
                              height="20"
                            />
                          </button>
                        </div>
                        <div v-else>
                          <button
                            class="btn btn-icon justify-content-start me-2"
                            v-if="candidateDetail[columnKey] != null"
                            @click="viewFileModal(candidateDetail[columnKey])"
                          >
                            <em
                              class="bx bxs-file bx-sm"
                              style="color:#2517d8"
                            ></em>
                          </button>
                          <button
                            class="btn btn-icon justify-content-start me-2"
                            v-else
                          >
                            <em class="bx bx-file-blank bx-sm"></em>
                          </button>
                        </div>
                      </span>

                      <div v-else class="col-6">
                        <span
                          data-bs-toggle="tooltip"
                          data-bs-placement="top"
                          :data-bs-title="
                            getCandidateDetail(
                              candidateDetail[columnKey],
                              columnKey,
                              column.type
                            )
                          "
                          data-bs-custom-class="custom-tooltip"
                        >
                          {{
                            $truncate(
                              getCandidateDetail(
                                candidateDetail[columnKey],
                                columnKey,
                                column.type
                              ),
                              { length: 15 }
                            )
                          }}
                        </span>
                      </div>
                    </div>
                  </div>
                  <div class="col-6 margin-check" v-if="socialFieldsHasValue">
                    <div class="row">
                      <div class="col-6 fwt-bold d-flex align-items-center">
                        Social
                      </div>
                      <div class="col-6" v-if="socialFieldsHasValue">
                        <ul class="social-links mt-auto">
                          <li
                            v-if="
                              checkVisiblity('profilelinkedin') &&
                                candidateDetail.profilelinkedin
                            "
                          >
                            <a
                              id="sTest-candidateDLinkedinLink"
                              v-bind:href="
                                getUrlProtocol(candidateDetail.profilelinkedin)
                              "
                              target="_blank"
                              class="me-2"
                              rel="noopener noreferrer"
                              ><em
                                class="bx bxl-linkedin-square bx-sm"
                                style="color:#0077b5"
                              ></em
                            ></a>
                          </li>
                          <li
                            v-if="
                              checkVisiblity('profiletwitter') &&
                                candidateDetail.profiletwitter
                            "
                          >
                            <a
                              id="sTest-candidateDTwitterLink"
                              v-bind:href="
                                getUrlProtocol(candidateDetail.profiletwitter)
                              "
                              target="_blank"
                              class="me-2"
                              rel="noopener noreferrer"
                              ><em
                                class="bx bxl-twitter bx-sm"
                                style="color:#006dbf"
                              ></em
                            ></a>
                          </li>
                          <li
                            v-if="
                              checkVisiblity('profilefacebook') &&
                                candidateDetail.profilefacebook
                            "
                          >
                            <a
                              id="sTest-candidateDFbLink"
                              v-bind:href="
                                getUrlProtocol(candidateDetail.profilefacebook)
                              "
                              target="_blank"
                              class="me-2"
                              rel="noopener noreferrer"
                              ><em
                                class="bx bxl-facebook-circle bx-sm"
                                style="color:#3b5998"
                              ></em
                            ></a>
                          </li>
                          <li
                            v-if="
                              checkVisiblity('profilegithub') &&
                                candidateDetail.profilegithub
                            "
                          >
                            <a
                              id="sTest-candidateDGithubLink"
                              v-bind:href="
                                getUrlProtocol(candidateDetail.profilegithub)
                              "
                              target="_blank"
                              class="me-2"
                              rel="noopener noreferrer"
                              ><em
                                class="bx bxl-github bx-sm"
                                style="color:#040404"
                              ></em
                            ></a>
                          </li>
                          <li
                            v-if="
                              checkVisiblity('profilexing') &&
                                candidateDetail.profilexing
                            "
                          >
                            <a
                              id="sTest-candidateDXingLink"
                              v-bind:href="
                                getUrlProtocol(candidateDetail.profilexing)
                              "
                              target="_blank"
                              class="me-2"
                              rel="noopener noreferrer"
                              ><em
                                class="bx bxl-xing bx-sm"
                                style="color:#06989d"
                              ></em
                            ></a>
                          </li>
                        </ul>
                      </div>
                    </div>
                  </div>
                  <div
                    v-if="allVisibleFieldsLength % 2 != 0"
                    class="col-6 margin-check"
                  >
                    <div class="row">
                      <div class="col-12 fwt-bold invisible">
                        Placeholder text
                      </div>
                    </div>
                  </div>
                  <div
                    class="col-12 margin-check"
                    v-if="checkVisiblity('languageskills')"
                  >
                    <div class="row">
                      <div class="col-3 fwt-bold">
                        {{ candidateDetailsFields.languageskills }}
                      </div>
                      <div class="col-9">
                        <ul class="list-style-disc mb-0 ps-3">
                          <li
                            v-for="(language, columnKey) in languageSkillsList"
                            :key="columnKey"
                          >
                            {{ language }}
                          </li>
                        </ul>
                      </div>
                    </div>
                  </div>
                  <div
                    class="col-12 margin-check"
                    v-if="checkVisiblity('skill')"
                  >
                    <div class="row">
                      <div class="col-3 fwt-bold">
                        {{ candidateDetailsFields.skill }}
                      </div>
                      <div class="col-9">
                        <span v-if="!candidateDetail.skill">Not Available</span>
                        <span
                          v-else
                          class="badge rounded-pill stage-badge p-2 fs-12 fwt-bold me-2 my-1"
                          v-for="(skill, columnKey) in getCandidateDetail(
                            candidateDetail.skill,
                            'skill'
                          )"
                          :key="columnKey"
                          >{{ skill }}</span
                        >
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <CandidateWorkHistory
              v-if="
                checkVisiblity('work_history') &&
                  flagsmithData.workAndEducationFlag &&
                  !isfromInternalPipeline
              "
              class="mt-4"
              :candidateDetails="candidateDetail"
              isFromProfileView="true"
            ></CandidateWorkHistory>
            <CandidateEducationHistory
              v-if="
                checkVisiblity('education_history') &&
                  flagsmithData.workAndEducationFlag &&
                  !isfromInternalPipeline
              "
              class="mt-4"
              :candidateDetails="candidateDetail"
              isFromProfileView="true"
            ></CandidateEducationHistory>
            <!-- Job Information Card -->
            <div class="card mt-4">
              <div class="card-body">
                <h5 class="card-title">Job Information</h5>
                <div class="row d-flex align-items-center ps-2">
                  <div class="col-3 fwt-bold">
                    Job Name
                  </div>
                  <div class="col-9 fwt-bold">
                    <div
                      class="avatar-text-45 bg-job color-job me-2"
                      v-if="jobData.jobById.name"
                    >
                      <span>
                        {{ getAcronym(jobData.jobById.name) }}
                      </span>
                    </div>
                    {{ getCandidateDetail(jobData.jobById.name) }}
                  </div>
                </div>
                <hr />
                <!-- later in internal hiring pipeline -->
                <div class="row d-flex align-items-center ps-2">
                  <div class="col-3 fwt-bold">
                    {{
                      isPortalViewDisabled
                        ? "Hiring Manager(s)"
                        : "Collaborators(s)"
                    }}
                  </div>
                  <div
                    class="col-9 d-inline-flex align-items-center flex-wrap"
                    v-if="jobData.jobById.hiring_managers.length != 0"
                  >
                    <div
                      v-for="(hiringmanager, index) in jobData.jobById
                        .hiring_managers"
                      :key="index"
                      class="me-5 d-flex align-items-center mt-2"
                    >
                      <div class="me-2" v-if="hiringmanager.photo != null">
                        <img
                          :src="hiringmanager.profile_url"
                          class="rounded-circle"
                          width="40"
                          height="40"
                          :class="{ 'd-none': isLoading }"
                        />
                        <div
                          :class="{ 'spinner-border': isLoading }"
                          role="status"
                        >
                          <span class="visually-hidden">Loading...</span>
                        </div>
                      </div>
                      <div
                        v-else
                        class="avatar-text-45 bg-hiringManager color-hiringManager me-2"
                      >
                        <span>
                          {{ getAcronym(hiringmanager.name) }}
                        </span>
                      </div>
                      <div>
                        <span class="fwt-bold d-flex">{{
                          hiringmanager.name
                        }}</span>
                      </div>
                    </div>
                  </div>
                  <div v-else class="col-3">
                    Not Available
                  </div>
                </div>
                <hr />
                <div class="row d-flex align-items-center ps-2">
                  <div class="col-3 fwt-bold">
                    Submitted By
                  </div>
                  <div class="col-3">
                    {{
                      getCandidateDetail(candidateDetail.agency_contact_name)
                    }}
                  </div>
                  <div class="col-3 fwt-bold">
                    {{ candidateDetailsFields.candidateupdatedon }}
                  </div>
                  <div class="col-3">
                    {{
                      getCandidateDetail(
                        candidateDetail.candidateupdatedon,
                        "candidateupdatedon",
                        "date"
                      )
                    }}
                  </div>
                </div>
              </div>
            </div>
          </div>
          <!-- Candidate Feedback (Right Side) -->
          <div id="rightColumn" class="col-3 border-start">
            <div class="modal-header">
              <h5 class="modal-title">Candidate Feedback</h5>
              <button
                id="sTest_closeCandidateDetailMdl"
                type="button"
                class="btn-close"
                aria-label="Close"
                @click="closeCandidateDetailsModal"
              ></button>
            </div>
            <HiringStageFeedbackComponent
              :candidateDetails="candidateDetail"
              :fromInternalPipeline="isfromInternalPipeline"
              :visibilityHiringStage="visibilityInHiringStage"
              :vmsJobId="jobData.jobById.id"
              :kanbanType="kanbanType"
              @updateStageEventHappened="hiringStageChanged"
            >
            </HiringStageFeedbackComponent>
          </div>
        </div>
      </div>
    </div>
  </div>
  <LongTextDisplayModal
    v-if="showLongTextModal"
    :strModalTitle="strLongTextFieldName"
    :strModalBody="strLongTextFieldValue"
    @closeLongTextModal="showLongTextModal = false"
  ></LongTextDisplayModal>
  <HTMLViewer
    v-if="openCandidateSummaryModal == true"
    @closeHtmlModal="openCandidateSummaryModal = false"
    entity="summary"
  ></HTMLViewer>
</template>

<script>
import UIPageHeaderResponsive from "@/components/UI/UIPageHeaderResponsive.vue";
import HiringStageFeedbackComponent from "@/views/Candidates/HiringStageFeedbackComponent.vue";
import LongTextDisplayModal from "@/components/LongTextDisplayModal.vue";
import { useRecruitmentManager } from "@/composables/useRecruitmentManager";
import { useCandidates } from "@/composables/useCandidates";
import { useJobsStore } from "@/store/jobStore";
import { inject, onMounted, ref, reactive } from "vue";
import { computed } from "@vue/reactivity";
import { useAuthStore } from "@/store/auth_store";
import mixpanel from "mixpanel-browser";
import {
  MIXPANEL_ACTIONS,
  MIXPANEL_EVENTS_PROPERTIES
} from "@/helpers/mixpanel_constants";
import { ABILITY_TOKEN } from "@casl/vue";
import {
  abilities,
  accessControlModules
} from "@/helpers/AccessControl/accessControlConstants";
import CandidateWorkHistory from "@/views/Candidates/CandidateWorkHistory.vue";
import CandidateEducationHistory from "@/views/Candidates/CandidateEducationHistory.vue";
import HTMLViewer from "@/components/HTMLViewer.vue";

export default {
  components: {
    UIPageHeaderResponsive,
    HiringStageFeedbackComponent,
    LongTextDisplayModal,
    CandidateWorkHistory,
    CandidateEducationHistory,
    HTMLViewer
  },
  props: {
    fromInternalPipeline: {
      default: false
    },
    candidatesColumns: {
      default: {}
    },
    kanbanType: {
      type: String,
      default: null
    }
  },
  setup(props, context) {
    const globalData = inject("globalStore");
    const jobData = useJobsStore();
    const { getAcronym, getDate, dateFormat, isJson } = inject(
      "useGlobalFunctions"
    )();
    const { getImageUrl, presigned_url } = useRecruitmentManager();
    const { getCandidateNameAsPerVisibility } = useCandidates();
    const ability = inject(ABILITY_TOKEN);
    const authStore = useAuthStore();
    let candidateDetailsFields = ref({});
    let fields = ref({});
    let candidateFieldsVisibility = ref({});
    let isLoading = ref(true);
    let customColumns = ref([]);
    let jobCustomFields = ref([]);
    let metaDataForNonSharedCandidates = ref({});
    let showCustomColumnsForInternalPipeline = ref(false);
    let candidateDetail = jobData.jobCandidatesById.candidateData;
    let candidateHiringStageUpdated = ref(false);
    globalData.candidateActionSource = "Candidate Detail Page";
    let candidateName = getCandidateNameAsPerVisibility(
      candidateFieldsVisibility.value,
      jobData.jobCandidatesById.candidateData
    );
    candidateDetail["candidatename"] = candidateName;
    let fieldsToIgnore = [
      "profilelinkedin",
      "profilefacebook",
      "profiletwitter",
      "profilegithub",
      "profilexing",
      "emailid",
      "contactnumber",
      "skill",
      "languageskills",
      "work_history",
      "education_history",
      "select_checkbox",
      "actions"
    ];
    let fileTypes = [
      "resumefilename",
      "coverletter",
      "portfolio",
      "other_file_1",
      "other_file_2",
      "formatted_cv",
      "summary"
    ];
    const showLongTextModal = ref(false);
    const strLongTextFieldName = ref("");
    const strLongTextFieldValue = ref("");
    const openCandidateSummaryModal = ref(false);
    let flagsmithData = jobData.agencyPageData["flagsmithData"][0];

    const fieldDataFunctionMapping = {
      noticeperiod: days => (days ? `${days} Days` : "Not Available"),
      willingtorelocate: value => (value === 1 ? "Yes" : "No"),
      salaryType: value => (value ? getSalaryType(value) : "Not Available"),
      workexpyr: years => (years ? `${years} Years` : "0 Years"),
      relevantexperience: years => (years ? `${years} Years` : "0 Years"),
      genderid: value => (value ? getGenderValue(value) : "Not Available"),
      currentsalary: value =>
        value ? `${candidateDetail.symbol || ""} ${value}` : "Not Available",
      salaryexpectation: value =>
        value ? `${candidateDetail.symbol || ""} ${value}` : "Not Available",
      skill: value => (value ? value.split(",") : ["Not Available"])
    };

    const allVisibleFieldsLength = computed(() => {
      if (socialFieldsHasValue.value) {
        return Object.keys(allVisibleFields.value).length + 1;
      }
      return Object.keys(allVisibleFields.value).length;
    });

    function getSalaryType(value) {
      const salaryType = [
        "Monthly Salary",
        "Annual Salary",
        "Weekly Salary",
        "Daily Salary",
        "Hourly Salary"
      ];
      return salaryType[value - 1];
    }

    function getGenderValue(value) {
      const genderMapping = ["Male", "Female", "Not Available"];
      return genderMapping[value - 1];
    }

    const languageSkillsList = computed(() => {
      return candidateDetail.languageskills
        ? candidateDetail.languageskills.split(",")
        : [];
    });

    const allVisibleFields = computed(() => {
      //remove the fields which are not visible
      let visibleStandardFields = candidateColumns.columns.filter(column => {
        return (
          checkVisiblity(column.field) &&
          column.field != "select_checkbox" &&
          !fieldsToIgnore.includes(column.field)
        );
      });

      //except

      let visibleCustomColumns = getCustomColumns();
      let visibleJobCustomColumns =
        flagsmithData.jobAssociatedFieldsFlag && !props.fromInternalPipeline
          ? getJobCustomFields()
          : [];
      //concat custom columns with visible fields
      visibleCustomColumns = visibleCustomColumns.concat(
        visibleJobCustomColumns
      );

      if (!jobData.jobCandidatesById.candidateData.visibility) {
        setCustomDataInCandidateDetail(visibleCustomColumns);
      }

      //merge custom columns with visible fields
      let visibleFields = visibleStandardFields.concat(visibleCustomColumns);

      //except summary if props.fromInternalPipeline is true
      if (props.fromInternalPipeline) {
        visibleFields = visibleFields.filter(column => {
          return column.field != "summary";
        });
      }
      //sort based on order
      visibleFields = visibleFields.sort((a, b) =>
        a.order > b.order ? 1 : -1
      );

      let n = visibleFields.length;
      let inputList = visibleFields;
      let outputList = [];
      for (let i = 0; i < n / 2; i++) {
        outputList.push(inputList[i]);

        const secondHalfIndex = i + Math.ceil(n / 2);
        if (secondHalfIndex < n) {
          outputList.push(inputList[secondHalfIndex]);
        }
      }

      visibleFields = outputList;

      visibleFields.forEach(element => {
        fields.value[element.field] = {
          label: element.headerName,
          type: element.type ? element.type : element.column_type
        };
      });
      return fields.value;
    });

    const visibilityOfStage = computed(() => {
      return (
        checkVisiblity("candidatestatus") &&
        jobData.jobCandidatesById.candidateData.rcrm_hiring_stage_label !=
          undefined
      );
    });

    const socialFieldsHasValue = computed(() => {
      return (
        (candidateDetail.profilelinkedin &&
          !candidateFieldsVisibility.value.profilelinkedin) ||
        (candidateDetail.profiletwitter &&
          !candidateFieldsVisibility.value.profiletwitter) ||
        (candidateDetail.profilegithub &&
          !candidateFieldsVisibility.value.profilegithub) ||
        (candidateDetail.profilexing &&
          !candidateFieldsVisibility.value.profilexing) ||
        (candidateDetail.profilefacebook &&
          !candidateFieldsVisibility.value.profilefacebook)
      );
    });

    const isPortalViewDisabled = computed(() => {
      return ability.can(abilities.VIEW_PORTAL, accessControlModules.AGENCIES);
    });

    //setting reactive candidate status if changed from hiring feedback
    jobData.$patch(state => {
      state.candidateHiringStages.externalPipeline =
        state.jobCandidatesById.candidateData.rcrm_hiring_stage_label;
      state.candidateHiringStages.internalPipeline =
        state.jobCandidatesById.candidateData.vms_hiring_stage_label;
    });

    //getting candidate field value
    function getCandidateDetail(value, field, type = "") {
      if (fieldDataFunctionMapping[field]) {
        return fieldDataFunctionMapping[field](value);
      }
      if (!value) {
        return "Not Available";
      }
      if (type == "date") {
        return dateFormat(value);
      }
      return value;
    }

    function isCustomField(columnKey) {
      return (
        columnKey.includes("custcolumn") ||
        columnKey.includes("job_associated_cust_column_")
      );
    }

    //getting the utl protocol for redirection
    function getUrlProtocol(url) {
      let isProtocol = /\b(http|https)/.test(url);
      if (!isProtocol) {
        url = "https://" + url;
      }
      return url.replace(/\/$/, "");
    }

    function checkVisiblity(column) {
      if (candidateFieldsVisibility.value[column]) {
        return false;
      }
      return true;
    }

    function viewFileModal(value, modal) {
      globalData.showModal["fileViewModal"] = true;
      if (modal == "showCandidateResumeModal") {
        jobData.jobCandidatesById.candidatename = candidateName;
        jobData.jobCandidatesById.resumefilename = value;
        jobData.jobCandidatesById.modal = "showCandidateResumeModal";
      } else {
        jobData.jobCandidatesById.customFileValue = value;
        jobData.jobCandidatesById.modal = "showViewFileModal";
      }
    }

    let sharedFields = "";
    const candidatesData = require("@/entity_columns/candidates.js");

    if (jobData.jobCandidatesById.candidateData.visibility == 1) {
      for (const [key, value] of Object.entries(
        jobData.jobCandidatesById.candidateData
      )) {
        if (key.includes("custcolumn")) {
          customColumns.value.push(value);
        }
        if (key.includes("job_associated_cust_column_")) {
          jobCustomFields.value.push(value);
        }
      }

      sharedFields =
        jobData.jobCandidatesById.candidateData.sharecandidatefields;
    } else {
      customColumns.value =
        jobData.jobCandidatesById.candidateData.custom_field_rcrm_copy;
      sharedFields =
        jobData.jobCandidatesById.candidateData.sharecandidatefields;
      jobCustomFields.value =
        jobData.jobCandidatesById.candidateData.job_associated_custom_field_rcrm_copy;
    }

    metaDataForNonSharedCandidates.value = {
      customColumns: customColumns.value,
      sharedFields: sharedFields,
      jobAssociatedCustomFields: jobCustomFields.value
    };

    let candidateColumns = candidatesData.getCandidateColumns(
      0,
      metaDataForNonSharedCandidates.value
    );

    candidateColumns.columns.forEach(element => {
      candidateDetailsFields.value[element.field] = element.headerName;
      candidateFieldsVisibility.value[element.field] = element.hide;
    });

    function closeCandidateDetailsModal() {
      globalData.showModal["viewCandidateProfileModal"] = false;
      globalData.candidateActionSource = "Candidate List Page";
      context.emit(
        "closeCandidateDetailsModal",
        candidateHiringStageUpdated.value
      );
    }

    function setCustomDataInCandidateDetail(customFields) {
      customFields.forEach(element => {
        candidateDetail[element.field] = element;
      });
    }

    function getJobCustomFields() {
      let arrJobCustomFields = ref(
        metaDataForNonSharedCandidates.value.jobAssociatedCustomFields
      );
      let sharedFields = ref(metaDataForNonSharedCandidates.value.sharedFields);
      let visibleJobCustomFields = [];

      if (isJson(sharedFields.value) && typeof sharedFields.value == "string") {
        sharedFields.value = JSON.parse(sharedFields.value);
      }
      if (sharedFields.value && arrJobCustomFields.value != null) {
        const fieldVisiblity = ref(sharedFields.value);
        arrJobCustomFields.value.forEach(field => {
          if (
            field &&
            fieldVisiblity.value[field.column_name] &&
            field.column_name.includes("job_associated_cust_column")
          ) {
            if (fieldVisiblity.value[field.column_name].visible == 1) {
              field.headerName =
                fieldVisiblity.value[field.column_name].external_label;
              field.order = fieldVisiblity.value[field.column_name].order;
              field.field = field.column_name;
              //push field name and field as object
              visibleJobCustomFields.push({ ...field });
            }
          }
        });
      }
      return visibleJobCustomFields;
    }

    function getCustomColumns() {
      let arrCustomColumns = ref(
        metaDataForNonSharedCandidates.value.customColumns
      );
      let sharedFields = ref(metaDataForNonSharedCandidates.value.sharedFields);
      let visibleCustomColumns = [];

      if (isJson(sharedFields.value) && typeof sharedFields.value == "string") {
        sharedFields.value = JSON.parse(sharedFields.value);
      }
      if (sharedFields.value && arrCustomColumns.value != null) {
        const fieldVisiblity = ref(sharedFields.value);
        arrCustomColumns.value.forEach(field => {
          if (
            field &&
            fieldVisiblity.value[field.column_name] &&
            field.column_name.includes("custcolumn")
          ) {
            if (fieldVisiblity.value[field.column_name].visible == 1) {
              field.headerName =
                fieldVisiblity.value[field.column_name].external_label;
              field.order = fieldVisiblity.value[field.column_name].order;
              field.field = field.column_name;
              //push field name and field as object
              visibleCustomColumns.push({ ...field });
            }
          }
        });
      }
      return visibleCustomColumns;
    }

    //get url for hiring manager profile image
    onMounted(async () => {
      isLoading.value = true;
      //using for loop to get presigned url for jobData.jobById.hiring_managers
      await Promise.all(
        jobData.jobById.hiring_managers.map(async manager => {
          if (manager.photo) {
            await getImageUrl(manager.photo, "job_details");
            manager.profile_url = presigned_url.value;
          } else {
            manager.profile_url = null;
          }
        })
      );

      isLoading.value = false;
      mixpanel.track(MIXPANEL_EVENTS_PROPERTIES.PAGE, {
        [MIXPANEL_ACTIONS.PAGE_NAME]: "Candidate Details",
        ...authStore.getMixpanelDefaultProperties()
      });
    });
    function hiringStageChanged() {
      candidateHiringStageUpdated.value = true;
    }

    const openLongTextModal = column => {
      strLongTextFieldName.value = column.headerName;
      strLongTextFieldValue.value = column.column_value;
      showLongTextModal.value = true;
    };

    const viewSummaryModal = () => {
      openCandidateSummaryModal.value = true;
    };

    return {
      globalData,
      isfromInternalPipeline: props.fromInternalPipeline,
      jobData,
      candidateDetail: jobData.jobCandidatesById.candidateData,
      candidateDetailsFields,
      getCandidateDetail,
      getAcronym,
      getUrlProtocol,
      checkVisiblity,
      getImageUrl,
      isLoading,
      presigned_url,
      getDate,
      dateFormat,
      customColumns,
      showCustomColumnsForInternalPipeline,
      metaDataForNonSharedCandidates,
      viewFileModal,
      viewSummaryModal,
      visibilityInHiringStage: {
        externalPipeline: checkVisiblity("candidatestatus"),
        remarkVisibility: checkVisiblity("remark"),
        isAddedToInternalPipeline:
          jobData.jobCandidatesById.addedToInternalPipeline
      },

      visibilityOfStage,
      socialFieldsHasValue,
      abilities,
      accessControlModules,
      ability,
      closeCandidateDetailsModal,
      candidateHiringStageUpdated,
      hiringStageChanged,
      authStore,
      isPortalViewDisabled,
      allVisibleFields,
      isCustomField,
      showLongTextModal,
      strLongTextFieldName,
      strLongTextFieldValue,
      openLongTextModal,
      allVisibleFieldsLength,
      fileTypes,
      openCandidateSummaryModal,
      languageSkillsList,
      flagsmithData
    };
  }
};
</script>

<style lang="scss" scoped>
.stage-badge {
  background-color: #eaf4ff;
  color: #3a6a88;
}

#pageHeader {
  padding: 1.5rem 0rem !important;
  border-top: none !important;
}

#leftColumn {
  overflow: scroll !important;
  position: absolute;
  top: 0px;
  bottom: 0;
  left: 0;
  overflow-y: scroll;
  background-color: #f5f4f4;
}

#rightColumn {
  height: 100%;
  position: absolute;
  top: 0;
  bottom: 0;
  right: 0;
  overflow-y: scroll;
}
.margin-check {
  border-bottom: 1px solid #d2d7da;
  line-height: 3.5rem;
}
</style>
