<template>
    <div class="modal fade" :class="['modal-mask', 'show', 'd-block']" tabindex="-1" aria-labelledby="exampleModalLabel"
        aria-hidden="true">
        <div class="modal-dialog h-100 mw-80" style="max-width: 90%">
            <div class="modal-content modal-dialog-scrollable">
                <div class="row justify-content-center align-items-center m-0">
                    <!-- Candidate Detail (Left Side) -->
                    <div id="leftColumn" class="col-7 mb-4">
                        <UIPageHeaderResponsive>
                            <template #preHeaderActions>
                                <div class="me-2"></div>
                                <div class="me-2">
                                <img
                                    class="image avatar-text-45"
                                    v-if="candidateDetail?.profilepic || candidateDetail?.profilepicUrl"
                                    :src="candidateDetail?.profilepicUrl?.split(',')[0] || candidateDetail?.profilepic?.split(',')[0]"
                                    alt=""
                                >
                                    <div v-else
                                        class="avatar-text-45 bg-candidate color-candidate">
                                        <span>
                                            {{ getAcronym(candidateDetail.candidatename) }}
                                        </span>
                                    </div>
                                </div>
                            </template>
                            <template #pageHeader>
                                <div class="row">
                                    <span class="fwt-bold fs-16">{{ candidateDetail.candidatename
                                    }}</span>
                                    <div class="fs-12 fwt-bold">
                                        <span class="text-muted d-none">
                                            Submitted 2 days ago
                                        </span>
                                    </div>
                                </div>
                            </template>
                            <template #pageActionsRight>
                                <div class="d-flex justify-content-end">
                                    <div class="text-end me-2 d-grid" v-if="visibilityOfStage">
                                        <span class="text-muted fwt-bold mb-2">Agency Hiring Stage </span>
                                        <span v-if="checkVisiblity('candidatestatus')"
                                            class="badge rounded-pill stage-badge p-2 fs-12 fwt-bold">{{
                                                    jobData.candidateHiringStages.externalPipeline
                                            }}</span>
                                    </div>
                                    <div class="vr me-2" v-if="visibilityOfStage && isPortalViewDisabled"></div>
                                    <div v-if="isPortalViewDisabled">
                                        <div class="d-grid" v-if="jobData.jobCandidatesById.addedToInternalPipeline">
                                            <span class="text-muted fwt-bold mb-2">Internal Hiring Stage</span>
                                            <span class="badge rounded-pill stage-badge p-2 fs-12 fwt-bold">
                                                {{jobData.candidateHiringStages.internalPipeline}}
                                            </span>
                                        </div>
                                        <div class="mt-2" v-else>
                                            <button
                                                v-if="ability.can(abilities.UPDATE, accessControlModules.CANDIDATES)"
                                                id="sTest_mapToJobBtn"
                                                class="btn btn-icon-text border fwt-bold"
                                                @click="addToInternalPipeline">
                                                <em class='bx bx-plus me-1'></em>
                                                Add to Internal Pipeline
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </template>
                        </UIPageHeaderResponsive>
                        <!-- Job Information Card -->
                        <div class="card mt-4" >
                            <div class="card-body">
                                <h5 class="card-title">Job Information</h5>
                                <div class="row d-flex align-items-center">
                                    <div class="col-3 fwt-bold">
                                        Job Name
                                    </div>
                                    <div class="col-9 fwt-bold">
                                        <div class="avatar-text-45 bg-job color-job me-2"
                                            v-if="jobData.jobById.name">
                                            <span>
                                                {{ getAcronym(jobData.jobById.name) }}
                                            </span>
                                        </div>
                                        {{ getCandidateDetail(jobData.jobById.name) }}
                                    </div>
                                </div>
                                <hr />
                                <!-- later in internal hiring pipeline -->
                                <div class="row d-flex align-items-center">
                                    <div class="col-3 fwt-bold">
                                       {{ isPortalViewDisabled ? 'Hiring Manager(s)' : 'Collaborators(s)'}}
                                    </div>
                                    <div class="col-9 d-inline-flex align-items-center flex-wrap" v-if="jobData.jobById.hiring_managers.length!=0">
                                        <div v-for="(hiringmanager, index) in jobData.jobById.hiring_managers" :key="index" class="me-5 d-flex align-items-center mt-2">
                                            <div class="me-2" v-if="(hiringmanager.photo != null)">
                                                <img :src="hiringmanager.profile_url" class="rounded-circle" width="40" height="40" :class="{'d-none':isLoading}" />
                                                <div :class="{'spinner-border':isLoading}" role="status" >
                                                    <span class="visually-hidden">Loading...</span>
                                                </div>
                                            </div>
                                            <div v-else class="avatar-text-45 bg-hiringManager color-hiringManager me-2">
                                                <span>
                                                    {{ getAcronym(hiringmanager.name) }}
                                                </span>
                                            </div>
                                            <div>
                                                <span class="fwt-bold d-flex">{{hiringmanager.name}}</span>
                                            </div>
                                        </div>
                                    </div>
                                    <div v-else class="col-3">
                                        Not Available
                                    </div>
                        
                                </div>
                            </div>
                        </div>
                        <!-- Agency Information Card -->
                        <div class="card mt-4">
                            <div class="card-body">
                                <div class="row d-flex align-items-center">
                                    <div class="col-3 fwt-bold" v-if="isPortalViewDisabled">
                                        Agency Name
                                    </div>
                                    <div class="col-3" v-if="isPortalViewDisabled">
                                        {{ getCandidateDetail(candidateDetail.agency_name)  }}
                                    </div>
                                    <div class="col-3 fwt-bold">
                                        Submitted By
                                    </div>
                                    <div class="col-3"> 
                                        {{ getCandidateDetail(candidateDetail.agency_contact_name) }}
                                    </div>
                                </div>
                                <hr />
                                <div class="row d-flex align-items-center">
                                    <div class="col-3 fwt-bold">
                                        {{ candidateDetailsFields.candidateupdatedon }}
                                    </div>
                                    <div class="col-3">
                                        {{ getCandidateDetail(candidateDetail.candidateupdatedon ,'candidateupdatedon') }}
                                    </div>
                                    <div class="col-3 fwt-bold" v-if="isPortalViewDisabled">
                                        Department
                                    </div>
                                    <div class="col-3" v-if="isPortalViewDisabled">
                                        {{ jobData.jobById.department_name?jobData.jobById.department_name:'Not Added' }}
                                    </div>
                                </div>
                            </div>
                        </div>
                        <!-- Candidate's Contact Information Card -->
                        <div class="card mt-4" v-if="objCandidateDetailsSections.isCandidateContactInformationSectionVisible.visible || objCandidateDetailsSections.isSocialLinksSectionVisible.visible">
                            <div class="card-body">
                                <div class="row d-flex align-items-center">
                                    <div class="col-3 fwt-bold d-flex align-items-center"
                                        v-if="checkVisiblity('emailid')">
                                        <em class="bx bxs-envelope text-muted me-2 bx-sm"></em>
                                        {{ candidateDetailsFields.emailid }}
                                    </div>
                                    <div class="col-3" v-if="checkVisiblity('emailid')">
                                        {{ getCandidateDetail(candidateDetail.emailid) }}
                                    </div>
                                    <div class="col-3 fwt-bold d-flex align-items-center"
                                        v-if="checkVisiblity('contactnumber')">
                                        <em class="bx bxs-phone-call text-muted me-2 bx-sm"></em>
                                        {{ candidateDetailsFields.contactnumber }}
                                    </div>
                                    <div class="col-3" v-if="checkVisiblity('contactnumber')">
                                        {{ getCandidateDetail(candidateDetail.contactnumber) }}
                                    </div>
                                </div>
                                <hr
                                    :class="{ 'd-none': ((!checkVisiblity('emailid') && !checkVisiblity('contactnumber')) || !objCandidateDetailsSections.isSocialLinksSectionVisible.visible )}" />
                                <div class="row d-flex align-items-center" v-if="objCandidateDetailsSections.isSocialLinksSectionVisible.visible">
                                    <div class="col-3 fwt-bold d-flex align-items-center">
                                        <em class="bx bx-share-alt text-muted me-2 bx-sm"></em> Social
                                    </div>
                                    <div class="col-3" v-if="socialFieldsHasValue">
                                        <ul class="social-links">
                                            <li v-if="checkVisiblity('profilelinkedin') && candidateDetail.profilelinkedin"><a id="sTest-candidateDLinkedinLink"
                                                v-bind:href="getUrlProtocol(candidateDetail.profilelinkedin)" target="_blank"
                                                class="me-2" rel="noopener noreferrer"><em class="bx bxl-linkedin-square bx-sm" style="color:#0077b5"></em></a>
                                            </li>
                                            <li v-if="checkVisiblity('profiletwitter') && candidateDetail.profiletwitter"><a id="sTest-candidateDTwitterLink"
                                                v-bind:href="getUrlProtocol(candidateDetail.profiletwitter)" target="_blank"
                                                class="me-2" rel="noopener noreferrer"><em class="bx bxl-twitter bx-sm" style="color:#006dbf"  ></em></a>
                                            </li>
                                            <li v-if="checkVisiblity('profilefacebook') && candidateDetail.profilefacebook"><a id="sTest-candidateDFbLink"
                                                v-bind:href="getUrlProtocol(candidateDetail.profilefacebook)" target="_blank"
                                                class="me-2" rel="noopener noreferrer"><em class="bx bxl-facebook-circle bx-sm" style="color:#3b5998" ></em></a>
                                            </li>
                                            <li v-if="checkVisiblity('profilegithub') && candidateDetail.profilegithub"><a id="sTest-candidateDGithubLink"
                                                v-bind:href="getUrlProtocol(candidateDetail.profilegithub)" target="_blank"
                                                class="me-2" rel="noopener noreferrer"><em class="bx bxl-github bx-sm" style="color:#040404"></em></a>
                                            </li>
                                            <li v-if="checkVisiblity('profilexing') && candidateDetail.profilexing"><a id="sTest-candidateDXingLink"
                                                v-bind:href="getUrlProtocol(candidateDetail.profilexing)" target="_blank"
                                                class="me-2" rel="noopener noreferrer"><em class="bx bxl-xing bx-sm" style="color:#06989d"  ></em></a>
                                            </li>
                                        </ul>
                                    </div>
                                    <div v-else class="col-3">
                                        Not Available
                                    </div>
                                    
                                </div>
                            </div>
                        </div>
                        <!-- Candidate's Personal Information Card -->
                        <div class="card mt-4" v-if="objCandidateDetailsSections.isCandidatePersonalInformationSectionVisible.visible">
                            <div class="card-body">
                                <div class="row d-flex align-items-center">
                                    <div class="col-3 fwt-bold" v-if="checkVisiblity('candidate_city')">
                                        {{ candidateDetailsFields.candidate_city }}
                                    </div>
                                    <div class="col-3" v-if="checkVisiblity('candidate_city')">
                                        {{ getCandidateDetail(candidateDetail.candidate_city) }}
                                    </div>
                                    <div class="col-3 fwt-bold" v-if="checkVisiblity('candidate_locality')">
                                        {{ candidateDetailsFields.candidate_locality }}
                                    </div>
                                    <div class="col-3" v-if="checkVisiblity('candidate_locality')">
                                        {{ getCandidateDetail(candidateDetail.candidate_locality) }}
                                    </div>
                                </div>
                                <hr
                                    :class="{ 'd-none': !checkVisiblity('candidate_city') && !checkVisiblity('candidate_locality') }" />
                                <div class="row d-flex align-items-center">
                                    <div class="col-3 fwt-bold" v-if="checkVisiblity('candidatedob')">
                                        {{ candidateDetailsFields.candidatedob }}
                                    </div>
                                    <div class="col-3" v-if="checkVisiblity('candidatedob')">
                                        {{ getDate(getCandidateDetail(candidateDetail.candidatedob)) }}
                                    </div>
                                    <div class="col-3 fwt-bold" v-if="checkVisiblity('willingtorelocate')">
                                        {{ candidateDetailsFields.willingtorelocate }}
                                    </div>
                                    <div class="col-3" v-if="checkVisiblity('willingtorelocate')">
                                        {{ getCandidateDetail(candidateDetail.willingtorelocate == 1 ? 'Yes' : 'No') }}
                                    </div>
                                </div>
                            </div>
                        </div>
                        <!-- Profile Field's Card -->
                        <div class="card mt-4" v-if="objCandidateDetailsSections.isProfileFieldsSectionVisible.visible">
                            <div class="card-body">
                                <h5 class="card-title pb-3">Profile Fields</h5>
                                <div class="row d-flex align-items-center">
                                    <div class="col-3 fwt-bold" v-if="checkVisiblity('languageskills')">
                                        {{ candidateDetailsFields.languageskills }}
                                    </div>
                                    <div class="col-9" v-if="checkVisiblity('languageskills')">
                                        {{  getCandidateDetail(candidateDetail.languageskills) }}
                                    </div>
                                </div>
                                <hr :class="{ 'd-none': !checkVisiblity('languageskills') }" />
                                <div class="row d-flex align-items-center">
                                    <div class="col-3 fwt-bold" v-if="checkVisiblity('skill')">
                                        {{ candidateDetailsFields.skill }}
                                    </div>
                                    <div class="col-9 fwt-bold" v-if="checkVisiblity('skill')">
                                        <span class="badge rounded-pill stage-badge p-2 fs-12 fwt-bold me-2 my-1"
                                            v-for="(skill, index) in skills" :key="index">{{ skill }}</span>
                                    </div>
                                </div>
                                <hr :class="{ 'd-none': !checkVisiblity('skill') }" />
                                <div class="row d-flex align-items-center">
                                    <span class="col-3 fwt-bold" v-if="checkVisiblity('qualification')">
                                        {{ candidateDetailsFields.qualification }}
                                    </span>
                                    <span class="col-9 " v-if="checkVisiblity('qualification')">
                                        {{ getCandidateDetail(candidateDetail.qualification) }}
                                    </span>
                                </div>
                                <hr :class="{ 'd-none': !checkVisiblity('qualification') }" />
                                <div class="row d-flex align-items-center">
                                    <span class="col-3 fwt-bold" v-if="checkVisiblity('specialization')">
                                        {{ candidateDetailsFields.specialization }}
                                    </span>
                                    <span class="col-9 " v-if="checkVisiblity('specialization')">
                                        {{ getCandidateDetail(candidateDetail.specialization) }}
                                    </span>
                                </div>
                            </div>
                        </div>
                        <!-- Employment Information Card -->
                        <div class="card mt-4" v-if="objCandidateDetailsSections.isEmploymentInformationSectionVisibile.visible">
                            <div class="card-body">
                                <h5 class="card-title pb-3">Employment Information</h5>
                                <div class="row d-flex align-items-center">
                                    <span class="col-3 fwt-bold" v-if="checkVisiblity('workexpyr')">
                                        {{ candidateDetailsFields.workexpyr }}
                                    </span>
                                    <span class="col-3" v-if="checkVisiblity('workexpyr')">
                                        {{ getCandidateDetail(candidateDetail.workexpyr, 'Years') }}
                                    </span>
                                    <span class="col-3 fwt-bold" v-if="checkVisiblity('relevantexperience')">
                                        {{ candidateDetailsFields.relevantexperience }}
                                    </span>
                                    <span class="col-3" v-if="checkVisiblity('relevantexperience')">
                                        {{ getCandidateDetail(candidateDetail.relevantexperience, 'Years') }}
                                    </span>
                                </div>
                                <hr
                                    :class="{ 'd-none': !checkVisiblity('workexpyr') && !checkVisiblity('relevantexperience') }" />

                                <div class="row d-flex align-items-center">
                                    <span class="col-3 fwt-bold" >
                                        Salary Type
                                    </span>
                                    <span class="col-3" >
                                        {{ getCandidateDetail(candidateDetail.salarytype, 'salaryType') }}
                                    </span>
                                    <span class="col-3 fwt-bold" v-if="checkVisiblity('lastorganisation')">
                                        {{ candidateDetailsFields.lastorganisation }}
                                    </span>
                                    <span class="col-3" v-if="checkVisiblity('lastorganisation')">
                                        {{ getCandidateDetail(candidateDetail.lastorganisation) }}
                                    </span>
                                </div>
                                <hr/>
                                <div v-if="checkVisiblity('salaryexpectation')" class="row d-flex align-items-center">
                                    <span class="col-3 fwt-bold" >
                                        Salary Expectation
                                    </span>
                                    <span class="col-3" >
                                        {{ (candidateDetail.salaryexpectation != 0 ? getCandidateDetail(candidateDetail.symbol) : '') + getCandidateDetail(candidateDetail.salaryexpectation) }}
                                    </span>
                                </div>
                                <hr :class="{ 'd-none': !checkVisiblity('salaryexpectation') }" />

								<div class="row d-flex align-items-center">
									<span class="col-3 fwt-bold" v-if="checkVisiblity('currentsalary')">
										{{ candidateDetailsFields.currentsalary }}
									</span>
									<span class="col-3" v-if="checkVisiblity('currentsalary')">
										{{ (candidateDetail.currentsalary != 0 ? getCandidateDetail(candidateDetail.symbol) : '') + getCandidateDetail(candidateDetail.currentsalary) }}
									</span>
									<span class="col-3 fwt-bold" v-if="checkVisiblity('availablefrom')">
										{{ candidateDetailsFields.availablefrom }}
									</span>
									<span class="col-3" v-if="checkVisiblity('availablefrom')">
										{{ getDate(getCandidateDetail(candidateDetail.availablefrom)) }}
									</span>
								</div>
								<hr
									:class="{ 'd-none': !checkVisiblity('currentsalary') && !checkVisiblity('availablefrom') }" />
								<div class="row d-flex align-items-center">
									<span class="col-3 fwt-bold" v-if="checkVisiblity('noticeperiod')">
										{{ candidateDetailsFields.noticeperiod }}
									</span>
									<span class="col-3" v-if="checkVisiblity('noticeperiod')">
										{{ getCandidateDetail(candidateDetail.noticeperiod, 'Days') }}
									</span>
								</div>
							</div>
						</div>
						<!-- Files Card -->
						<div class="card mt-4" v-if="objCandidateDetailsSections.isFilesSectionVisible.visible">
							<div class="card-body">
								<h5 class="card-title pb-3">Files</h5>
								<div class="row d-flex align-items-center">
									<span class="col-3 fwt-bold" v-if="checkVisiblity('resumefilename')">
										Resume
									</span>
									<span class="col-9 d-flex align-items-center" v-if="checkVisiblity('resumefilename')">
										<button class='btn btn-icon me-2' v-if="candidateDetail.resumefilename != null" @click="viewFileModal(candidateDetail.resumefilename,'showCandidateResumeModal')" ><em class='bx bxs-file bx-sm' style="color:#2517d8"></em></button>
										<button class='btn btn-icon me-2' v-else ><em class='bx bx-file-blank bx-sm'></em></button> 
									</span>
                                    <hr :class="{ 'd-none': !checkVisiblity('resumefilename') }"/>
                                    <span class="col-3 fwt-bold" v-if="checkVisiblity('coverletter')">
										Job Application Cover Letter
									</span>
									<span class="col-9 d-flex align-items-center"  v-if="checkVisiblity('coverletter')">
										<button class='btn btn-icon me-2' v-if="candidateDetail.coverletter != null" @click="viewFileModal(candidateDetail.coverletter)" ><em class='bx bxs-file bx-sm' style="color:#2517d8"></em></button>
										<button class='btn btn-icon me-2' v-else ><em class='bx bx-file-blank bx-sm'></em></button> 
									</span>
                                    <hr :class="{ 'd-none': !checkVisiblity('coverletter') }"/>
                                    <span class="col-3 fwt-bold" v-if="checkVisiblity('formatted_cv')">
                                        Job Application Resume
                                    </span>
                                    <span class="col-9 d-flex align-items-center" v-if="checkVisiblity('formatted_cv')">
                                        <button class='btn btn-icon me-2' v-if="candidateDetail.formatted_cv != null" @click="viewFileModal(candidateDetail.formatted_cv)" ><em class='bx bxs-file bx-sm' style="color:#2517d8"></em></button>
                                        <button class='btn btn-icon me-2' v-else ><em class='bx bx-file-blank bx-sm'></em></button>
                                    </span>
                                    <hr :class="{ 'd-none': !checkVisiblity('formatted_cv') }"/>
                                    <span class="col-3 fwt-bold" v-if="checkVisiblity('other_file_1')">
                                        Job Application File 1
                                    </span>
                                    <span class="col-9 d-flex align-items-center" v-if="checkVisiblity('other_file_1')">
                                        <button class='btn btn-icon me-2' v-if="candidateDetail.other_file_1 != null" @click="viewFileModal(candidateDetail.other_file_1)" ><em class='bx bxs-file bx-sm' style="color:#2517d8"></em></button>
                                        <button class='btn btn-icon me-2' v-else ><em class='bx bx-file-blank bx-sm'></em></button>
                                    </span>
                                    <hr :class="{ 'd-none': !checkVisiblity('other_file_1') }"/>
                                    <span class="col-3 fwt-bold" v-if="checkVisiblity('other_file_2')">
                                        Job Application File 2
                                    </span>
                                    <span class="col-9 d-flex align-items-center" v-if="checkVisiblity('other_file_2')">
                                        <button class='btn btn-icon me-2' v-if="candidateDetail.other_file_2 != null" @click="viewFileModal(candidateDetail.other_file_2)" ><em class='bx bxs-file bx-sm' style="color:#2517d8"></em></button>
                                        <button class='btn btn-icon me-2' v-else ><em class='bx bx-file-blank bx-sm'></em></button>
                                    </span>
                                    <hr :class="{ 'd-none': !checkVisiblity('other_file_2') }"/>
                                    <span class="col-3 fwt-bold" v-if="checkVisiblity('portfolio')">
                                        Job Application Portfolio
                                    </span>
                                    <span class="col-9 d-flex align-items-center" v-if="checkVisiblity('portfolio')">
                                        <button class='btn btn-icon me-2' v-if="candidateDetail.portfolio != null" @click="viewFileModal(candidateDetail.portfolio)" ><em class='bx bxs-file bx-sm' style="color:#2517d8"></em></button>
                                        <button class='btn btn-icon me-2' v-else ><em class='bx bx-file-blank bx-sm'></em></button>
                                    </span>                                   
								</div>
							</div>
						</div>
						<!-- Additional Information-->
						<CustomFieldsComponent @showFileModal="viewFileModal" :metaData="metaDataForNonSharedCandidates" />
					</div>
					<!-- Candidate Feedback (Right Side) -->
					<div id="rightColumn" class="col-5 border-start">
						<div class="modal-header">
							<h5 class="modal-title">Candidate Feedback</h5>
							<button id="sTest_closeCandidateDetailMdl" type="button" class="btn-close"
								aria-label="Close"
								@click="closeCandidateDetailsModal"></button>
						</div>
						<HiringStageFeedbackComponent :candidateDetails="candidateDetail" :fromInternalPipeline="isfromInternalPipeline" :visibilityHiringStage="visibilityInHiringStage" :vmsJobId="jobData.jobById.id" :kanbanType="kanbanType" @updateStageEventHappened="hiringStageChanged">
						</HiringStageFeedbackComponent>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import UIPageHeaderResponsive from "@/components/UI/UIPageHeaderResponsive.vue";
import HiringStageFeedbackComponent from "@/views/Candidates/HiringStageFeedbackComponent.vue";
import { useRecruitmentManager } from "@/composables/useRecruitmentManager";
import { useCandidates } from "@/composables/useCandidates";
import { useJobsStore } from "@/store/jobStore";
import { inject, onMounted, ref ,reactive} from "vue";
import { computed } from "@vue/reactivity";
import CustomFieldsComponent from '@/views/Candidates/CustomFieldsComponent.vue'
import { useAuthStore } from '@/store/auth_store';
import mixpanel from "mixpanel-browser";
import { MIXPANEL_ACTIONS, MIXPANEL_EVENTS_PROPERTIES } from "@/helpers/mixpanel_constants";
import { ABILITY_TOKEN } from '@casl/vue';
import { abilities, accessControlModules } from '@/helpers/AccessControl/accessControlConstants';

export default {
	components: {
		UIPageHeaderResponsive,
		HiringStageFeedbackComponent,
        CustomFieldsComponent,
	},
	props     : {
		fromInternalPipeline     : {
			default: false
		},
		candidatesColumns: {
			default :{}
		},
        kanbanType : {
            type: String,
            default: null
        },
    },
    setup(props,context) {
        const globalData = inject("globalStore");
        const jobData = useJobsStore();
        const { getAcronym, getDate, dateFormat } = inject("useGlobalFunctions")();
        const { getImageUrl, presigned_url } = useRecruitmentManager();
        const { getCandidateNameAsPerVisibility } = useCandidates();
        const ability = inject(ABILITY_TOKEN);
        const authStore = useAuthStore();
        let candidateDetailsFields = ref({});
        let candidateFieldsVisibility = ref({});
        let isLoading = ref(true);
        let customColumns = ref([]);
        let metaDataForNonSharedCandidates = ref({});
        let showCustomColumnsForInternalPipeline = ref(false);
        let candidateDetail = jobData.jobCandidatesById.candidateData;
        let candidateHiringStageUpdated = ref(false);
        globalData.candidateActionSource = 'Candidate Detail Page';
        let candidateName = getCandidateNameAsPerVisibility(candidateFieldsVisibility.value, jobData.jobCandidatesById.candidateData);
        candidateDetail['candidatename'] = candidateName;
        
        
        const salaryType = [
            { id: "2", label: "Annual Salary" },
            { id: "1", label: "Monthly Salary" },
            { id: "3", label: "Weekly Salary" },
            { id: "4", label: "Daily Salary" },
            { id: "5", label: "Hourly Salary" }
        ];

		
        const visibilityExternalPipeline = computed(() => {
            const { addedToInternalPipeline, candidateData } = jobData.jobCandidatesById;
            const hasHiringStageLabel = candidateData.rcrm_hiring_stage_label !== undefined;

            return addedToInternalPipeline && hasHiringStageLabel && checkVisiblity('candidatestatus');
        });

        const visibilityOfStage = computed(() => {
            return (checkVisiblity('candidatestatus') && jobData.jobCandidatesById.candidateData.rcrm_hiring_stage_label!=undefined)
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
            return ability.can(abilities.VIEW_PORTAL, accessControlModules.AGENCIES)
        });

        //setting reactive candidate status if changed from hiring feedback
        jobData.$patch((state) => {
            state.candidateHiringStages.externalPipeline = state.jobCandidatesById.candidateData.rcrm_hiring_stage_label;
            state.candidateHiringStages.internalPipeline = state.jobCandidatesById.candidateData.vms_hiring_stage_label;
        });

        //convert comma separed skill to array
        const candidateSkills = jobData.jobCandidatesById.candidateData.skill || "Not Available";
        const skills = candidateSkills.split(",");

        //getting candidate field value
        function getCandidateDetail(candidateData, extraInfo = "") {
            if (!candidateData) {
                return "Not Available";
            }
            let result;

            switch (extraInfo) {
                case "salaryType": {
                    const objSalaryType = salaryType.find(type => type.id === candidateData);
                    result = objSalaryType ? objSalaryType.label : "Not Available";
                    break;
                }
                case "fileType": {
                    const customField = candidateData;
                    result = jobData.jobCandidatesById.candidateData[customField] || null;
                    break;
                }
                case "candidateupdatedon":
                    result = dateFormat(candidateData);
                    break;
                default:
                    result = candidateData + " " + extraInfo;
                    break;
            }

            return result;
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

        function addToInternalPipeline(){
            globalData.showModal['confirmationModal']=true;
            jobData.candidateToInternalPipeline = [jobData.jobCandidatesById.candidateData];
            mixpanel.track(MIXPANEL_ACTIONS.ADD_TO_INTERNAL_PIPELINE_BTN_CLICKED, authStore.getMixpanelDefaultProperties());
        }
        
		function viewFileModal(value, modal) {
			globalData.showModal["fileViewModal"] = true;
            if(modal == 'showCandidateResumeModal'){
                jobData.jobCandidatesById.candidatename = candidateName;
                jobData.jobCandidatesById.resumefilename = value;
                jobData.jobCandidatesById.modal = 'showCandidateResumeModal';
            }else{
                jobData.jobCandidatesById.customFileValue = value;
			    jobData.jobCandidatesById.modal = 'showViewFileModal';
            }
        }

        let sharedFields = "";
        const candidatesData = require("@/entity_columns/candidates.js");

        if(jobData.jobCandidatesById.candidateData.visibility == 1) {

            for (const [key, value] of Object.entries(jobData.jobCandidatesById.candidateData)) {
                if(key.includes('custcolumn')) {
                    customColumns.value.push(value);
                }
            }

            sharedFields = jobData.jobCandidatesById.candidateData.sharecandidatefields;

        } else {

            customColumns.value = jobData.jobCandidatesById.candidateData.custom_field_rcrm_copy;
            sharedFields = jobData.jobCandidatesById.candidateData.sharecandidatefields;

        }

        metaDataForNonSharedCandidates.value = {
            'customColumns': customColumns.value,
            'sharedFields': sharedFields
        }


        let candidateColumns = candidatesData.getCandidateColumns(0, metaDataForNonSharedCandidates.value);

        candidateColumns.columns.forEach(element => {
            candidateDetailsFields.value[element.field] = element.headerName
            candidateFieldsVisibility.value[element.field] = element.hide
        });

        let objCandidateDetailsSections = reactive({
            isSocialLinksSectionVisible : {fields : ['profilelinkedin','profilefacebook','profiletwitter','profilegithub','profilexing'],visible:true},
            isCandidateContactInformationSectionVisible : {fields : ['emailid','contactnumber'],visible:true},
            isCandidatePersonalInformationSectionVisible : {fields : ['candidate_city','candidate_locality','candidatedob','willingtorelocate'],visible:true},
            isEmploymentInformationSectionVisibile : {fields : ['workexpyr','salaryType','lastorganisation','relevantexperience'],visible:true},
            isProfileFieldsSectionVisible : {fields : ['skill','qualification','specialization','languageskills'],visible:true},
            isFilesSectionVisible : {fields : ['other_file_2','other_file_1','portfolio','resumefilename','coverletter','formatted_cv'],visible:true},
        })

        Object.entries(objCandidateDetailsSections).forEach(([section, sectionData]) => {
            let visibility = false;
            sectionData.fields.forEach((field) => {
                visibility = visibility || !candidateFieldsVisibility.value[field] 
            })  
            sectionData.visible = visibility;
        })


        function closeCandidateDetailsModal () {
            globalData.showModal['viewCandidateProfileModal'] = false; 
            globalData.candidateActionSource = 'Candidate List Page';
            context.emit('closeCandidateDetailsModal',candidateHiringStageUpdated.value);
        }

        //get url for hiring manager profile image
        onMounted(async () => {
            isLoading.value = true;
            //using for loop to get presigned url for jobData.jobById.hiring_managers
            await Promise.all(jobData.jobById.hiring_managers.map(async manager => {
                if (manager.photo) {
                    await getImageUrl(manager.photo, "job_details");
                    manager.profile_url = presigned_url.value;
                } else {
                    manager.profile_url = null;
                }
            }));

            isLoading.value = false;
            mixpanel.track(MIXPANEL_EVENTS_PROPERTIES.PAGE,{
                [MIXPANEL_ACTIONS.PAGE_NAME] : 'Candidate Details',
                ...authStore.getMixpanelDefaultProperties()
            });
        });
        function hiringStageChanged(){
		    candidateHiringStageUpdated.value = true;
	    }
       
        return {
            globalData,
            isfromInternalPipeline :props.fromInternalPipeline,
            jobData,
            candidateDetail       : jobData.jobCandidatesById.candidateData,
            skills,
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
            addToInternalPipeline,
            visibilityInHiringStage : { 
                                    'externalPipeline': checkVisiblity('candidatestatus'),
                                    'remarkVisibility': checkVisiblity('remark'),
                                    'isAddedToInternalPipeline':jobData.jobCandidatesById.addedToInternalPipeline,
                                },      
            visibilityExternalPipeline,
            visibilityOfStage,
            socialFieldsHasValue,
            objCandidateDetailsSections,
            abilities,
            accessControlModules,
            ability,
            closeCandidateDetailsModal,
            candidateHiringStageUpdated,
            hiringStageChanged,
            authStore,
            isPortalViewDisabled
        };
    }
};
</script>

<style lang="scss" scoped>
.stage-badge {
    background-color: #EAF4FF;
    color: #3A6A88;
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
</style>

