<template>
  <div class="diagnostic-exam-page">
    <!-- Exam Header -->
    <div class="exam-header" v-if="!examStarted && exam">
      <div class="container">
        <h1>{{ exam.title }}</h1>
        <p>Diagnostic Case Examination</p>
      </div>
    </div>
    
    <!-- Loading indicator -->
    <div class="exam-loading" v-if="!exam && !examStarted && !examCompleted">
      <div class="container text-center py-5">
        <div class="spinner-border text-primary" role="status">
          <span class="visually-hidden">Loading...</span>
        </div>
        <p class="mt-3">Loading diagnostic exam data...</p>
      </div>
    </div>
    
    <!-- Pre-Exam Information -->
    <div class="exam-info" v-if="!examStarted && !examCompleted && exam">
      <div class="container">
        <div class="exam-card">
          <div class="row">
            <div class="col-md-8">
              <h2>Exam Overview</h2>
              <p class="lead">{{ exam.description }}</p>
              
              <div class="exam-details">
                <div class="detail-item">
                  <div class="detail-icon">
                    <i class="bi bi-clock"></i>
                  </div>
                  <div class="detail-content">
                    <h4>Duration</h4>
                    <p>{{ exam.duration }}</p>
                  </div>
                </div>
                
                <div class="detail-item">
                  <div class="detail-icon">
                    <i class="bi bi-question-circle"></i>
                  </div>
                  <div class="detail-content">
                    <h4>Cases</h4>
                    <p>{{ diagnosticCases?.length || 0 }} clinical cases to analyze</p>
                  </div>
                </div>
              </div>
              
              <div class="exam-instructions alert alert-info">
                <h3><i class="bi bi-info-circle"></i> Exam Instructions</h3>
                <ol>
                  <li><strong>Clinical Reasoning Assessment:</strong> This is a diagnostic case examination designed to test your clinical reasoning and diagnostic skills.</li>
                  <li><strong>Case Presentation:</strong> Each case includes:
                    <ul>
                      <li>Patient demographics and clinical history</li>
                      <li>High-quality diagnostic images (X-rays, CT scans, MRI, etc.)</li>
                      <li>Relevant laboratory results when applicable</li>
                    </ul>
                  </li>
                  <li><strong>Navigation:</strong> 
                    <ul>
                      <li>Use the "Next Case" and "Previous Case" buttons to navigate between cases</li>
                      <li>You can review and change your answers at any time during the exam</li>
                      <li>A progress indicator shows your current position in the exam</li>
                    </ul>
                  </li>
                  <li><strong>Image Viewing:</strong>
                    <ul>
                      <li>Click on any image to view it in full screen</li>
                      <li>Use the zoom controls to examine details</li>
                      <li>Multiple images may be provided for comprehensive assessment</li>
                    </ul>
                  </li>
                  <li><strong>Time Management:</strong> You have {{ exam.duration }} to complete all cases. A timer will display the remaining time.</li>
                  <li><strong>Submission:</strong> After reviewing all cases, click "Submit Exam" to complete your attempt. You'll receive immediate feedback on your performance.</li>
                  <li><strong>Important:</strong> Once submitted, you cannot modify your answers for this attempt.</li>
                </ol>
                
                <div class="alert alert-warning mt-3 mb-0">
                  <i class="bi bi-exclamation-triangle"></i> <strong>Note:</strong> Ensure you have a stable internet connection throughout the exam. Your progress is automatically saved as you navigate between cases.
                </div>
              </div>
              
              <div class="d-flex justify-content-between mt-4">
                <a :href="getBackLink()" class="btn btn-secondary">
                  <i class="bi bi-arrow-left"></i> Back
                </a>
                <button 
                  class="btn btn-primary" 
                  @click="beginExam"
                >
                  Start Exam <i class="bi bi-arrow-right"></i>
                </button>
              </div>
            </div>
            
            <div class="col-md-4">
              <div class="exam-summary">
                <h3>Exam Information</h3>
                
                <div class="performance-stats">
                  <h4>Previous Attempts</h4>
                  <div v-if="previousAttempts.length > 0">
                    <div 
                      v-for="(attempt, index) in previousAttempts" 
                      :key="index" 
                      class="stats-item"
                    >
                      <div class="stats-label">{{ attempt.date }}</div>
                      <div class="stats-value">{{ attempt.score }}%</div>
                    </div>
                    <div class="stats-item mt-3">
                      <div class="stats-label">Best Score</div>
                      <div class="stats-value">{{ bestScore }}%</div>
                    </div>
                  </div>
                  <div v-else class="text-muted">
                    No previous attempts
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    
    <!-- Diagnostic Exam Content -->
    <div class="exam-content" v-if="examStarted && !examCompleted">
      <div class="container">
        <div class="diagnostic-interface">
          <div class="row">
            <!-- Diagnostic Case Panel -->
            <div class="col-md-9">
              <div class="case-panel">
                <div class="case-header">
                  <h2>{{ currentCase?.title || 'Clinical Case' }}</h2>
                  <div class="case-navigation">
                    <span class="case-counter">Case {{ currentCaseIndex + 1 }} of {{ diagnosticCases.length }}</span>
                  </div>
                </div>
                
                <div class="case-content">
                  <!-- Student Instructions Section -->
                  <div v-if="currentCase?.instructions" class="student-instructions mb-4">
                    <div class="alert alert-warning">
                      <h4><i class="bi bi-exclamation-triangle"></i> Instructions for this Case</h4>
                      <div v-html="currentCase.instructions"></div>
                    </div>
                  </div>
                  
                  <!-- Clinical Information Section -->
                  <div class="case-description">
                    <h3>Clinical Information</h3>
                    <div class="clinical-info p-3 mb-3" v-html="getClinicalInfo()"></div>
                  </div>
                  
                  <!-- Presenting Symptoms Section -->
                  <div v-if="currentCase?.presenting_symptoms" class="presenting-symptoms mb-4">
                    <h4><i class="bi bi-clipboard2-pulse"></i> Presenting Symptoms</h4>
                    <div class="symptoms-box p-3" v-html="currentCase.presenting_symptoms"></div>
                  </div>
                  
                  <div class="diagnostic-images mb-4" ref="imageContainer" v-if="getMainImageUrl() || getAdditionalImages().length > 0">
                    <div class="image-container">
                      <!-- Main image with overlay and zoom icon -->
                      <div v-if="getMainImageUrl()" class="main-image-container mb-4">
                        <div class="image-card">
                          <div class="image-header">
                            <h5><i class="bi bi-image"></i> Primary Diagnostic Image</h5>
                            <button class="btn-zoom" @click="showLightbox(getMainImageUrl())" title="View full size">
                              <i class="bi bi-zoom-in"></i>
                            </button>
                          </div>
                          <div class="image-wrapper">
                            <img 
                              :src="getMainImageUrl()" 
                              class="main-image" 
                              alt="Primary diagnostic image"
                              @error="handleImageError"
                              @load="handleImageLoad"
                            >
                            <div class="image-overlay" @click="showLightbox(getMainImageUrl())">
                              <span><i class="bi bi-search"></i> Click to enlarge</span>
                            </div>
                          </div>
                        </div>
                      </div>
                      
                      <!-- Additional images with thumbnails -->
                      <div v-if="getAdditionalImages().length > 0" class="additional-images-container">
                        <h5 class="mb-3"><i class="bi bi-images"></i> Additional Images ({{ getAdditionalImages().length }})</h5>
                        <div class="image-gallery">
                          <div 
                            v-for="(img, idx) in getAdditionalImages()" 
                            :key="idx" 
                            class="gallery-item"
                          >
                            <div class="thumbnail-wrapper">
                              <img 
                                :src="img" 
                                class="img-fluid thumbnail" 
                                alt="Additional diagnostic image"
                                @error="handleImageError"
                                @load="handleImageLoad"
                              >
                              <div class="thumbnail-overlay" @click="showLightbox(img)">
                                <i class="bi bi-search"></i>
                              </div>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                  
                  <!-- Lightbox Modal for Image Viewing -->
                  <div class="lightbox-modal" v-if="lightboxActive" @click="hideLightbox">
                    <div class="lightbox-content" @click.stop>
                      <button class="lightbox-close" @click="hideLightbox">
                        <i class="bi bi-x-lg"></i>
                      </button>
                      
                      <!-- Navigation buttons -->
                      <button 
                        v-if="lightboxImages.length > 1" 
                        class="lightbox-nav lightbox-prev" 
                        @click.stop="prevLightboxImage"
                        aria-label="Previous image"
                      >
                        <i class="bi bi-chevron-left"></i>
                      </button>
                      
                      <button 
                        v-if="lightboxImages.length > 1" 
                        class="lightbox-nav lightbox-next" 
                        @click.stop="nextLightboxImage"
                        aria-label="Next image"
                      >
                        <i class="bi bi-chevron-right"></i>
                      </button>
                      
                      <div class="lightbox-image-container">
                        <img :src="lightboxImage" class="lightbox-image" alt="Enlarged diagnostic image">
                      </div>
                      
                      <!-- Image counter for multiple images -->
                      <div v-if="lightboxImages.length > 1" class="lightbox-counter">
                        {{ lightboxIndex + 1 }} / {{ lightboxImages.length }}
                      </div>
                    </div>
                  </div>
                  
                  <div class="diagnostic-questions mb-4">
                    <h3>Questions</h3>
                    
                    <!-- No questions message when needed -->
                    <div v-if="!currentCaseQuestions || currentCaseQuestions.length === 0" class="alert alert-info">
                      No questions available for this diagnostic case.
                    </div>
                    
                    <!-- Questions list -->
                    <div 
                      v-for="(question, qIndex) in currentCaseQuestions" 
                      :key="question.id || qIndex" 
                      class="question-item p-3 mb-3"
                    >
                      <div class="question-text mb-3">{{ question.text }}</div>
                      
                      <!-- Multiple choice options -->
                      <div v-if="question.type === 'multiple_choice' || question.question_type === 'multiple_choice'" class="options-list">
                        <div 
                          v-for="(option, oIndex) in question.options" 
                          :key="oIndex" 
                          class="form-check mb-2"
                        >
                          <input 
                            :id="`option-${qIndex}-${oIndex}`" 
                            class="form-check-input" 
                            type="radio" 
                            :name="`question-${question.id || qIndex}`" 
                            :checked="userCaseAnswers[question.id] === oIndex"
                            @change="setAnswer(question.id || qIndex, oIndex)"
                          >
                          <label 
                            class="form-check-label" 
                            :for="`option-${qIndex}-${oIndex}`"
                          >
                            {{ option.text }}
                          </label>
                        </div>
                      </div>
                      
                      <!-- Short answer / text response -->
                      <div v-else class="mb-3">
                        <textarea 
                          class="form-control" 
                          rows="3" 
                          placeholder="Enter your answer here..."
                          v-model="userCaseAnswers[question.id || qIndex]"
                        ></textarea>
                      </div>
                    </div>
                  </div>
                  
                  <div class="case-navigation-buttons">
                    <button 
                      class="btn btn-secondary me-2" 
                      @click="previousCase"
                      :disabled="currentCaseIndex === 0"
                    >
                      <i class="bi bi-arrow-left"></i> Previous Case
                    </button>
                    <button 
                      v-if="currentCaseIndex < diagnosticCases.length - 1"
                      class="btn btn-primary" 
                      @click="nextCase"
                      :disabled="examCompleted"
                    >
                      Next Case <i class="bi bi-arrow-right"></i>
                    </button>
                    <button 
                      v-if="currentCaseIndex >= diagnosticCases.length - 1 && !examCompleted"
                      class="btn btn-success" 
                      @click="finishDiagnosticExam"
                    >
                      Complete Exam <i class="bi bi-check-circle"></i>
                    </button>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- Exam Navigation Panel -->
            <div class="col-md-3">
              <div class="navigation-panel">
                <div class="navigation-header">
                  <h3>Exam Navigation</h3>
                </div>
                
                <div class="navigation-status">
                  <div class="status-item">
                    <div class="status-value">{{ diagnosticCases.length }}</div>
                    <div class="status-label">Total Cases</div>
                  </div>
                  <div class="status-item">
                    <div class="status-value" :class="{'text-danger': remainingTime < 300}">
                      {{ formatTime(remainingTime) }}
                    </div>
                    <div class="status-label">Time Left</div>
                  </div>
                </div>
                
                <div class="case-progress mb-4">
                  <div class="progress mb-2">
                    <div 
                      class="progress-bar bg-success" 
                      role="progressbar" 
                      :style="`width: ${(currentCaseIndex / diagnosticCases.length) * 100}%`" 
                      :aria-valuenow="(currentCaseIndex / diagnosticCases.length) * 100" 
                      aria-valuemin="0" 
                      aria-valuemax="100"
                    ></div>
                  </div>
                  <div class="small text-muted text-center">
                    {{ currentCaseIndex }} of {{ diagnosticCases.length }} cases completed
                  </div>
                </div>
                
                <div class="case-list">
                  <h4 class="mb-3">Case List</h4>
                  <div class="list-group">
                    <button
                      v-for="(caseItem, index) in diagnosticCases"
                      :key="caseItem.id"
                      class="list-group-item list-group-item-action d-flex justify-content-between align-items-center"
                      :class="{'active': index === currentCaseIndex}"
                      @click="goToCase(index)"
                    >
                      <span>Case {{ index + 1 }}: {{ caseItem.title }}</span>
                      <span v-if="caseAnswered(index)" class="badge bg-success rounded-pill">
                        <i class="bi bi-check"></i>
                      </span>
                    </button>
                  </div>
                </div>
                
                <div class="navigation-footer mt-4">
                  <button 
                    class="btn btn-danger w-100" 
                    data-bs-toggle="modal" 
                    data-bs-target="#endExamModal"
                  >
                    End Exam <i class="bi bi-x-circle"></i>
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- End Exam Confirmation Modal -->
    <div class="modal fade" id="endExamModal" tabindex="-1" aria-labelledby="endExamModalLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="endExamModalLabel">End Exam</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <p>Are you sure you want to end the exam? This action cannot be undone.</p>
            <div class="alert alert-warning">
              <i class="bi bi-exclamation-triangle-fill"></i> All unanswered questions will be marked as incorrect.
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="button" class="btn btn-danger" @click="endExam">End Exam</button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Exam Results Section -->
    <div class="exam-results" v-if="examCompleted">
      <div class="container">
        <div class="results-card">
          <div v-if="diagnosticResults" class="exam-results-header mb-4">
            <div class="alert alert-info">
              <h3 class="mb-3"><i class="bi bi-check-circle-fill me-2"></i>Diagnostic Exam Completed</h3>
              <div class="row">
                <div class="col-md-4">
                  <div class="score-display">
                    <div class="score-circle" :class="getScoreClass(diagnosticResults.totalScore)">
                      <div class="score-value">{{ diagnosticResults.totalScore }}%</div>
                    </div>
                    <div class="score-label">Overall Score</div>
                  </div>
                </div>
                <div class="col-md-8">
                  <div class="results-summary">
                    <div class="row">
                      <div class="col-md-6">
                        <p><strong>Total Questions:</strong> {{ diagnosticResults.totalQuestions }}</p>
                        <p><strong>Correct Answers:</strong> {{ diagnosticResults.correctCount }}</p>
                        <p><strong>Cases Reviewed:</strong> {{ diagnosticResults.caseResults.length }}</p>
                        <p><strong>Completed:</strong> {{ new Date(diagnosticResults.completedAt).toLocaleString() }}</p>
                      </div>
                      <div class="col-md-6">
                        <p><strong>Pass Threshold:</strong> {{ exam.passmark || 70 }}%</p>
                        <p>
                          <strong>Result:</strong> 
                          <span 
                            :class="diagnosticResults.totalScore >= (exam.passmark || 70) ? 'text-success' : 'text-danger'"
                          >
                            {{ diagnosticResults.totalScore >= (exam.passmark || 70) ? 'PASS' : 'NEEDS IMPROVEMENT' }}
                          </span>
                        </p>
                        <p><strong>Time Spent:</strong> {{ formatTime(exam.duration_minutes * 60 - remainingTime) }}</p>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          
          <!-- Enhanced Exam Completion Banner -->
          <div v-if="examCompleted && diagnosticResults" class="exam-completion-banner mb-4">
            <div class="completion-card">
              <div class="completion-header">
                <div class="completion-icon">
                  <i class="bi bi-trophy-fill"></i>
                </div>
                <div class="completion-content">
                  <h2 class="completion-title">Exam Completed Successfully!</h2>
                  <p class="completion-subtitle">Excellent work! You've finished the diagnostic examination.</p>
                </div>
              </div>
              
              <div class="completion-stats">
                <div class="stat-item">
                  <div class="stat-value">{{ diagnosticResults.totalQuestions }}</div>
                  <div class="stat-label">Questions</div>
                </div>
                <div class="stat-item">
                  <div class="stat-value">{{ diagnosticResults.correctCount }}</div>
                  <div class="stat-label">Correct</div>
                </div>
                <div class="stat-item">
                  <div class="stat-value">{{ Math.round(diagnosticResults.percentage) }}%</div>
                  <div class="stat-label">Score</div>
                </div>
                <div class="stat-item">
                  <div class="stat-value">{{ diagnosticCases.length }}</div>
                  <div class="stat-label">Cases</div>
                </div>
              </div>
              
              <div class="completion-footer">
                <div class="performance-indicator" :class="getPerformanceClass(diagnosticResults.percentage)">
                  <i :class="getPerformanceIcon(diagnosticResults.percentage)"></i>
                  <span>{{ getPerformanceText(diagnosticResults.percentage) }}</span>
                </div>
                <p class="review-note">Review your answers below to learn from detailed explanations and improve your diagnostic skills.</p>
              </div>
            </div>
          </div>
          
          <!-- Individual case review -->
          <div class="case-panel mb-4">
            <div class="case-header">
              <h3>
                {{ currentCase?.title || 'Clinical Case' }}
                <span v-if="examCompleted && diagnosticResults" class="badge bg-info ms-2">Case {{ currentCaseIndex + 1 }}</span>
              </h3>
              <div class="case-navigation">
                <span class="case-counter">Case {{ currentCaseIndex + 1 }} of {{ diagnosticCases.length }}</span>
              </div>
            </div>
            
            <div class="case-content">
              <!-- Student Instructions Section -->
              <div v-if="currentCase?.instructions" class="student-instructions mb-4">
                <div class="alert alert-warning">
                  <h4><i class="bi bi-exclamation-triangle"></i> Instructions for this Case</h4>
                  <div v-html="currentCase.instructions"></div>
                </div>
              </div>
              
              <!-- Clinical Information Section -->
              <div class="case-description">
                <h3>Clinical Information</h3>
                <div class="clinical-info p-3 mb-3" v-html="getClinicalInfo()"></div>
              </div>
              
              <!-- Presenting Symptoms Section -->
              <div v-if="currentCase?.presenting_symptoms" class="presenting-symptoms mb-4">
                <h4><i class="bi bi-clipboard2-pulse"></i> Presenting Symptoms</h4>
                <div class="symptoms-box p-3" v-html="currentCase.presenting_symptoms"></div>
              </div>
              
              <div v-if="examCompleted && diagnosticResults" class="case-results-summary mb-3">
                <div class="alert alert-light border">
                  <h5>Case Results</h5>
                  <div v-if="getCurrentCaseResult()">
                    <p><strong>Score for this case:</strong> 
                      <span :class="{'text-success': getCurrentCaseResult().score >= 70, 'text-danger': getCurrentCaseResult().score < 70}">
                        {{ getCurrentCaseResult().score }}%
                      </span>
                    </p>
                    <p><strong>Questions:</strong> {{ getCurrentCaseResult().totalQuestions }}</p>
                    <p><strong>Correct:</strong> {{ getCurrentCaseResult().correctCount }}</p>
                  </div>
                  <div v-else>
                    <p>No results available for this case.</p>
                  </div>
                </div>
              </div>
              
              <div class="diagnostic-images mb-4" v-if="getMainImageUrl() || getAdditionalImages().length > 0">
                <div class="image-container">
                  <!-- Main image with overlay and zoom icon -->
                  <div v-if="getMainImageUrl()" class="main-image-container mb-4">
                    <div class="image-card">
                      <div class="image-header">
                        <h5><i class="bi bi-image"></i> Primary Diagnostic Image</h5>
                        <button class="btn-zoom" @click="showLightbox(getMainImageUrl())" title="View full size">
                          <i class="bi bi-zoom-in"></i>
                        </button>
                      </div>
                      <div class="image-wrapper">
                        <img 
                          :src="getMainImageUrl()" 
                          class="main-image" 
                          alt="Primary diagnostic image"
                          @error="handleImageError"
                          @load="handleImageLoad"
                        >
                        <div class="image-overlay" @click="showLightbox(getMainImageUrl())">
                          <span><i class="bi bi-search"></i> Click to enlarge</span>
                        </div>
                      </div>
                    </div>
                  </div>
                  
                  <!-- Additional images with thumbnails -->
                  <div v-if="getAdditionalImages().length > 0" class="additional-images-container">
                    <h5 class="mb-3"><i class="bi bi-images"></i> Additional Images ({{ getAdditionalImages().length }})</h5>
                    <div class="image-gallery">
                      <div 
                        v-for="(img, idx) in getAdditionalImages()" 
                        :key="idx" 
                        class="gallery-item"
                      >
                        <div class="thumbnail-wrapper">
                          <img 
                            :src="img" 
                            class="img-fluid thumbnail" 
                            alt="Additional diagnostic image"
                            @error="handleImageError"
                            @load="handleImageLoad"
                          >
                          <div class="thumbnail-overlay" @click="showLightbox(img)">
                            <i class="bi bi-search"></i>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
              
              <div class="diagnostic-questions-review mb-5" :style="{
                background: 'linear-gradient(180deg, #ffffff 0%, #f8fafb 100%)',
                borderRadius: '24px',
                boxShadow: '0 20px 60px rgba(0, 0, 0, 0.08)',
                overflow: 'visible',
                marginTop: '40px',
                position: 'relative',
                border: '2px solid rgba(102, 126, 234, 0.1)'
              }">
                <div class="premium-section-header" :style="{
                  position: 'relative',
                  background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
                  padding: '40px',
                  overflow: 'hidden',
                  borderRadius: '24px 24px 0 0'
                }">
                  <div class="header-gradient-overlay" :style="{
                    position: 'absolute',
                    inset: '0',
                    background: 'radial-gradient(circle at top right, rgba(16, 185, 129, 0.2) 0%, transparent 50%), radial-gradient(circle at bottom left, rgba(139, 92, 246, 0.2) 0%, transparent 50%)',
                    pointerEvents: 'none'
                  }"></div>
                  <div class="header-content" :style="{
                    position: 'relative',
                    zIndex: '1',
                    display: 'flex',
                    justifyContent: 'space-between',
                    alignItems: 'center',
                    gap: '30px'
                  }">
                    <div class="section-title-group" :style="{
                      display: 'flex',
                      alignItems: 'center',
                      gap: '24px'
                    }">
                      <div class="icon-wrapper" :style="{
                        position: 'relative',
                        width: '64px',
                        height: '64px'
                      }">
                        <div class="icon-glow" :style="{
                          position: 'absolute',
                          inset: '-8px',
                          background: 'radial-gradient(circle, rgba(255, 255, 255, 0.3) 0%, transparent 70%)',
                          borderRadius: '50%',
                          animation: 'subtle-pulse 3s ease-in-out infinite'
                        }"></div>
                        <i class="bi bi-journal-medical" :style="{
                          width: '64px',
                          height: '64px',
                          background: 'rgba(255, 255, 255, 0.2)',
                          backdropFilter: 'blur(10px)',
                          border: '1px solid rgba(255, 255, 255, 0.3)',
                          borderRadius: '16px',
                          display: 'flex',
                          alignItems: 'center',
                          justifyContent: 'center',
                          fontSize: '28px',
                          color: 'white',
                          boxShadow: '0 8px 24px rgba(0, 0, 0, 0.1)'
                        }"></i>
                      </div>
                      <div class="title-text">
                        <h2 class="main-title" :style="{
                          fontSize: '2rem',
                          fontWeight: '800',
                          color: 'white',
                          margin: '0 0 4px 0',
                          letterSpacing: '-0.02em'
                        }">Case Questions & Review</h2>
                        <p class="subtitle" :style="{
                          fontSize: '1rem',
                          color: 'rgba(255, 255, 255, 0.8)',
                          margin: '0',
                          fontWeight: '400'
                        }">Comprehensive analysis and learning insights</p>
                      </div>
                    </div>
                    <div class="section-stats glass-card" :style="{
                      background: 'rgba(255, 255, 255, 0.1)',
                      backdropFilter: 'blur(20px)',
                      border: '1px solid rgba(255, 255, 255, 0.2)',
                      borderRadius: '16px',
                      padding: '16px 24px',
                      display: 'flex',
                      alignItems: 'center',
                      gap: '20px',
                      boxShadow: '0 8px 32px rgba(0, 0, 0, 0.1)'
                    }">
                      <div class="stat-item" :style="{ textAlign: 'center' }">
                        <span class="stat-value" :style="{
                          display: 'block',
                          fontSize: '1.75rem',
                          fontWeight: '800',
                          color: 'white',
                          lineHeight: '1',
                          marginBottom: '4px'
                        }">{{ currentCaseQuestions.length }}</span>
                        <span class="stat-label" :style="{
                          fontSize: '0.875rem',
                          color: 'rgba(255, 255, 255, 0.8)',
                          fontWeight: '500'
                        }">{{ currentCaseQuestions.length === 1 ? 'Question' : 'Questions' }}</span>
                      </div>
                      <div v-if="examCompleted && getCurrentCaseResult()" class="stat-divider" :style="{
                        width: '1px',
                        height: '40px',
                        background: 'rgba(255, 255, 255, 0.2)'
                      }"></div>
                      <div v-if="examCompleted && getCurrentCaseResult()" class="stat-item" :style="{ textAlign: 'center' }">
                        <span class="stat-value" :style="{
                          display: 'block',
                          fontSize: '1.75rem',
                          fontWeight: '800',
                          color: 'white',
                          lineHeight: '1',
                          marginBottom: '4px'
                        }">{{ getCurrentCaseResult().score }}%</span>
                        <span class="stat-label" :style="{
                          fontSize: '0.875rem',
                          color: 'rgba(255, 255, 255, 0.8)',
                          fontWeight: '500'
                        }">Score</span>
                      </div>
                    </div>
                  </div>
                </div>
                
                <!-- Premium Questions Container -->
                <div class="questions-wrapper" :style="{
                  padding: '0 32px 32px'
                }">
                  <div 
                    v-for="(question, qIndex) in currentCaseQuestions" 
                    :key="question.id || qIndex" 
                    class="premium-question-card"
                    :style="{
                      background: '#ffffff',
                      borderRadius: '20px',
                      padding: '32px',
                      marginBottom: '24px',
                      boxShadow: '0 10px 40px rgba(0, 0, 0, 0.06)',
                      border: '2px solid transparent',
                      backgroundClip: 'padding-box',
                      backgroundImage: 'linear-gradient(#ffffff, #ffffff), linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
                      backgroundOrigin: 'border-box',
                      backgroundClip: 'padding-box, border-box',
                      transition: 'all 0.3s cubic-bezier(0.4, 0, 0.2, 1)',
                      position: 'relative',
                      overflow: 'hidden'
                    }"
                  >
                    <!-- Modern Question Header -->
                    <div class="question-header-modern" :style="{
                      display: 'flex',
                      alignItems: 'center',
                      justifyContent: 'space-between',
                      paddingBottom: '20px',
                      borderBottom: '2px solid #f0f4f8',
                      marginBottom: '24px'
                    }">
                      <div class="question-number-section" :style="{
                        display: 'flex',
                        alignItems: 'center',
                        gap: '12px'
                      }">
                        <div class="number-circle" :style="{
                          background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
                          color: '#ffffff',
                          width: '40px',
                          height: '40px',
                          borderRadius: '50%',
                          display: 'flex',
                          alignItems: 'center',
                          justifyContent: 'center',
                          fontSize: '18px',
                          fontWeight: '700',
                          boxShadow: '0 4px 15px rgba(102, 126, 234, 0.3)'
                        }">
                          <span class="number">{{ qIndex + 1 }}</span>
                          <div class="number-glow-effect"></div>
                        </div>
                        <div class="question-line"></div>
                      </div>
                      <div class="question-info-section" :style="{
                        display: 'flex',
                        alignItems: 'center',
                        gap: '16px'
                      }">
                        <div class="question-badges" :style="{
                          display: 'flex',
                          gap: '12px'
                        }">
                          <span class="type-badge-modern" :style="{
                            background: getQuestionTypeName(question) === 'Clinical' ? 'linear-gradient(135deg, #10b981 0%, #059669 100%)' : 'linear-gradient(135deg, #3b82f6 0%, #2563eb 100%)',
                            color: '#ffffff',
                            padding: '6px 16px',
                            borderRadius: '20px',
                            fontSize: '13px',
                            fontWeight: '500',
                            display: 'flex',
                            alignItems: 'center',
                            gap: '6px',
                            boxShadow: '0 2px 10px ' + (getQuestionTypeName(question) === 'Clinical' ? 'rgba(16, 185, 129, 0.3)' : 'rgba(59, 130, 246, 0.3)')
                          }">
                            <i :class="getQuestionTypeIcon(question)"></i>
                            <span>{{ getQuestionTypeName(question) }}</span>
                          </span>
                          <span class="points-badge-modern" :style="{
                            background: 'linear-gradient(135deg, #f59e0b 0%, #d97706 100%)',
                            color: '#ffffff',
                            padding: '6px 16px',
                            borderRadius: '20px',
                            fontSize: '13px',
                            fontWeight: '500',
                            display: 'flex',
                            alignItems: 'center',
                            gap: '6px',
                            boxShadow: '0 2px 10px rgba(245, 158, 11, 0.3)'
                          }">
                            <i class="bi bi-star-fill"></i>
                            <span>{{ question.points || 10 }} pts</span>
                          </span>
                        </div>
                        <div v-if="examCompleted" class="result-indicator" :style="{
                          background: isQuestionCorrect(question) ? 'rgba(16, 185, 129, 0.1)' : 'rgba(239, 68, 68, 0.1)',
                          color: isQuestionCorrect(question) ? '#059669' : '#dc2626',
                          padding: '6px 16px',
                          borderRadius: '20px',
                          fontSize: '13px',
                          fontWeight: '600',
                          display: 'flex',
                          alignItems: 'center',
                          gap: '6px',
                          border: '1px solid ' + (isQuestionCorrect(question) ? 'rgba(16, 185, 129, 0.3)' : 'rgba(239, 68, 68, 0.3)')
                        }">
                          <i :class="isQuestionCorrect(question) ? 'bi bi-check-circle-fill' : 'bi bi-x-circle-fill'"></i>
                          <span>{{ isQuestionCorrect(question) ? 'Correct' : 'Incorrect' }}</span>
                        </div>
                      </div>
                    </div>
                    
                    <!-- Elegant Question Content -->
                    <div class="question-content-modern" :style="{
                      marginBottom: '28px'
                    }">
                      <p class="question-text" :style="{
                        fontSize: '18px',
                        lineHeight: '1.8',
                        color: '#1e293b',
                        fontWeight: '500',
                        margin: '0'
                      }">{{ question.text }}</p>
                    </div>
                  
                      <!-- Modern Options Design -->
                    <div v-if="question.type === 'multiple_choice' || question.question_type === 'multiple_choice'" class="options-modern-container" :style="{
                      display: 'flex',
                      flexDirection: 'column',
                      gap: '12px',
                      marginBottom: '24px'
                    }">
                      <div 
                        v-for="(option, oIndex) in question.options" 
                        :key="oIndex" 
                        class="option-modern-card" 
                        :style="{
                          padding: '16px 20px',
                          borderRadius: '12px',
                          border: '2px solid ' + (option.is_correct && examCompleted ? '#10b981' : (!option.is_correct && userCaseAnswers[question.id] === oIndex && examCompleted ? '#ef4444' : '#e5e7eb')),
                          background: option.is_correct && examCompleted ? 'linear-gradient(135deg, rgba(16, 185, 129, 0.1) 0%, rgba(5, 150, 105, 0.1) 100%)' : (!option.is_correct && userCaseAnswers[question.id] === oIndex && examCompleted ? 'linear-gradient(135deg, rgba(239, 68, 68, 0.1) 0%, rgba(220, 38, 38, 0.1) 100%)' : '#ffffff'),
                          cursor: 'pointer',
                          transition: 'all 0.2s ease',
                          position: 'relative',
                          overflow: 'hidden'
                        }"
                      >
                        <div class="option-modern-content" :style="{
                          display: 'flex',
                          alignItems: 'center',
                          gap: '12px'
                        }">
                          <div class="option-indicator-modern" :style="{
                            width: '32px',
                            height: '32px',
                            borderRadius: '50%',
                            display: 'flex',
                            alignItems: 'center',
                            justifyContent: 'center',
                            flexShrink: '0',
                            background: (examCompleted && option.is_correct) ? 'linear-gradient(135deg, #10b981 0%, #059669 100%)' : ((examCompleted && userCaseAnswers[question.id] === oIndex && !option.is_correct) ? 'linear-gradient(135deg, #ef4444 0%, #dc2626 100%)' : 'linear-gradient(135deg, #e5e7eb 0%, #d1d5db 100%)'),
                            color: (examCompleted && (option.is_correct || (userCaseAnswers[question.id] === oIndex && !option.is_correct))) ? '#ffffff' : '#6b7280',
                            fontSize: '14px',
                            fontWeight: '600'
                          }">
                            <div v-if="examCompleted && option.is_correct" class="indicator-icon success">
                              <i class="bi bi-check-lg"></i>
                            </div>
                            <div v-else-if="examCompleted && userCaseAnswers[question.id] === oIndex && !option.is_correct" class="indicator-icon error">
                              <i class="bi bi-x-lg"></i>
                            </div>
                            <div v-else class="indicator-letter">
                              {{ String.fromCharCode(65 + oIndex) }}
                            </div>
                          </div>
                          <div class="option-text-wrapper" :style="{
                            flex: '1'
                          }">
                            <p class="option-text" :style="{
                              fontSize: '16px',
                              lineHeight: '1.6',
                              color: (examCompleted && option.is_correct) ? '#059669' : ((examCompleted && userCaseAnswers[question.id] === oIndex && !option.is_correct) ? '#dc2626' : '#374151'),
                              fontWeight: (examCompleted && (option.is_correct || userCaseAnswers[question.id] === oIndex)) ? '500' : '400',
                              margin: '0'
                            }">{{ option.text }}</p>
                            <div v-if="examCompleted" class="option-feedback" :style="{
                              marginTop: '8px'
                            }">
                              <span v-if="option.is_correct" class="feedback-badge success" :style="{
                                fontSize: '12px',
                                color: '#059669',
                                fontWeight: '600',
                                display: 'flex',
                                alignItems: 'center',
                                gap: '4px'
                              }">
                                <i class="bi bi-check-circle"></i> Correct Answer
                              </span>
                              <span v-else-if="userCaseAnswers[question.id] === oIndex" class="feedback-badge info" :style="{
                                fontSize: '12px',
                                color: '#dc2626',
                                fontWeight: '600',
                                display: 'flex',
                                alignItems: 'center',
                                gap: '4px'
                              }">
                                <i class="bi bi-cursor"></i> Your Selection
                              </span>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  
                  <!-- Text answer display for completed exam -->
                  <div v-else-if="examCompleted" class="answer-review-section" :style="{
                    display: 'flex',
                    flexDirection: 'column',
                    gap: '20px',
                    marginBottom: '24px'
                  }">
                    <div class="answer-card user-answer" :style="{
                      background: 'linear-gradient(135deg, rgba(59, 130, 246, 0.05) 0%, rgba(37, 99, 235, 0.05) 100%)',
                      border: '2px solid rgba(59, 130, 246, 0.2)',
                      borderRadius: '16px',
                      padding: '24px',
                      position: 'relative',
                      overflow: 'hidden'
                    }">
                      <div class="answer-header" :style="{
                        display: 'flex',
                        alignItems: 'center',
                        justifyContent: 'space-between',
                        marginBottom: '16px',
                        paddingBottom: '12px',
                        borderBottom: '1px solid rgba(59, 130, 246, 0.15)'
                      }">
                        <div class="answer-title" :style="{
                          display: 'flex',
                          alignItems: 'center',
                          gap: '10px',
                          fontSize: '16px',
                          fontWeight: '600',
                          color: '#1e40af'
                        }">
                          <div :style="{
                            width: '32px',
                            height: '32px',
                            background: 'linear-gradient(135deg, #3b82f6 0%, #2563eb 100%)',
                            borderRadius: '50%',
                            display: 'flex',
                            alignItems: 'center',
                            justifyContent: 'center',
                            color: 'white',
                            boxShadow: '0 4px 12px rgba(59, 130, 246, 0.3)'
                          }">
                            <i class="bi bi-person-fill" :style="{ fontSize: '16px' }"></i>
                          </div>
                          Your Answer
                        </div>
                        <div class="answer-score">
                          <span v-if="isTextAnswerCorrect(question, userCaseAnswers[question.id])" class="score-badge excellent" :style="{
                            background: 'linear-gradient(135deg, #10b981 0%, #059669 100%)',
                            color: 'white',
                            padding: '6px 16px',
                            borderRadius: '20px',
                            fontSize: '13px',
                            fontWeight: '600',
                            display: 'flex',
                            alignItems: 'center',
                            gap: '6px',
                            boxShadow: '0 4px 12px rgba(16, 185, 129, 0.3)'
                          }">
                            <i class="bi bi-check-circle"></i> Excellent
                          </span>
                          <span v-else class="score-badge needs-improvement" :style="{
                            background: 'linear-gradient(135deg, #f59e0b 0%, #d97706 100%)',
                            color: 'white',
                            padding: '6px 16px',
                            borderRadius: '20px',
                            fontSize: '13px',
                            fontWeight: '600',
                            display: 'flex',
                            alignItems: 'center',
                            gap: '6px',
                            boxShadow: '0 4px 12px rgba(245, 158, 11, 0.3)'
                          }">
                            <i class="bi bi-exclamation-triangle"></i> Needs Improvement
                          </span>
                        </div>
                      </div>
                      <div class="answer-content" :style="{
                        fontSize: '16px',
                        lineHeight: '1.7',
                        color: '#1e40af',
                        fontWeight: '500',
                        padding: '16px',
                        background: 'rgba(255, 255, 255, 0.7)',
                        borderRadius: '12px',
                        border: '1px solid rgba(59, 130, 246, 0.1)',
                        minHeight: '60px',
                        whiteSpace: 'pre-wrap'
                      }">
                        {{ userCaseAnswers[question.id] || 'No answer provided' }}
                      </div>
                    </div>
                    
                    <div class="answer-card model-answer" :style="{
                      background: 'linear-gradient(135deg, rgba(16, 185, 129, 0.05) 0%, rgba(5, 150, 105, 0.05) 100%)',
                      border: '2px solid rgba(16, 185, 129, 0.2)',
                      borderRadius: '16px',
                      padding: '24px',
                      position: 'relative',
                      overflow: 'hidden'
                    }">
                      <div class="answer-header" :style="{
                        display: 'flex',
                        alignItems: 'center',
                        gap: '10px',
                        marginBottom: '16px',
                        paddingBottom: '12px',
                        borderBottom: '1px solid rgba(16, 185, 129, 0.15)'
                      }">
                        <div class="answer-title" :style="{
                          display: 'flex',
                          alignItems: 'center',
                          gap: '10px',
                          fontSize: '16px',
                          fontWeight: '600',
                          color: '#047857'
                        }">
                          <div :style="{
                            width: '32px',
                            height: '32px',
                            background: 'linear-gradient(135deg, #10b981 0%, #059669 100%)',
                            borderRadius: '50%',
                            display: 'flex',
                            alignItems: 'center',
                            justifyContent: 'center',
                            color: 'white',
                            boxShadow: '0 4px 12px rgba(16, 185, 129, 0.3)'
                          }">
                            <i class="bi bi-star-fill" :style="{ fontSize: '16px' }"></i>
                          </div>
                          Model Answer
                        </div>
                      </div>
                      <div class="answer-content" :style="{
                        fontSize: '16px',
                        lineHeight: '1.7',
                        color: '#047857',
                        fontWeight: '500',
                        padding: '16px',
                        background: 'rgba(255, 255, 255, 0.7)',
                        borderRadius: '12px',
                        border: '1px solid rgba(16, 185, 129, 0.1)',
                        minHeight: '60px',
                        whiteSpace: 'pre-wrap'
                      }">
                        {{ question.model_answer }}
                      </div>
                    </div>
                  </div>
                  
                    <!-- Premium Explanation Section -->
                    <div v-if="examCompleted && question.explanation" class="explanation-premium-card" :style="{
                      background: 'linear-gradient(135deg, rgba(251, 207, 232, 0.3) 0%, rgba(251, 113, 133, 0.2) 100%)',
                      borderRadius: '16px',
                      padding: '24px',
                      border: '2px solid rgba(251, 113, 133, 0.3)',
                      position: 'relative',
                      overflow: 'hidden',
                      marginTop: '24px'
                    }">
                      <div class="explanation-header-modern" :style="{
                        display: 'flex',
                        alignItems: 'center',
                        gap: '12px',
                        marginBottom: '16px'
                      }">
                        <div class="explanation-icon-wrapper" :style="{
                          position: 'relative',
                          width: '40px',
                          height: '40px',
                          display: 'flex',
                          alignItems: 'center',
                          justifyContent: 'center'
                        }">
                          <i class="bi bi-lightbulb-fill" :style="{
                            fontSize: '24px',
                            background: 'linear-gradient(135deg, #f59e0b 0%, #d97706 100%)',
                            WebkitBackgroundClip: 'text',
                            WebkitTextFillColor: 'transparent'
                          }"></i>
                          <div class="icon-pulse"></div>
                        </div>
                        <h4 class="explanation-title" :style="{
                          margin: '0',
                          fontSize: '18px',
                          fontWeight: '600',
                          color: '#d97706',
                          letterSpacing: '0.5px'
                        }">Learning Insight</h4>
                      </div>
                      <div class="explanation-content-modern" :style="{
                        marginBottom: '16px'
                      }">
                        <div class="explanation-text" v-html="question.explanation" :style="{
                          fontSize: '16px',
                          lineHeight: '1.8',
                          color: '#78350f',
                          fontWeight: '500'
                        }"></div>
                      </div>
                      <div class="explanation-footer" :style="{
                        borderTop: '1px solid rgba(251, 113, 133, 0.2)',
                        paddingTop: '12px',
                        marginTop: '16px'
                      }">
                        <span class="tip-label" :style="{
                          fontSize: '13px',
                          color: '#92400e',
                          display: 'flex',
                          alignItems: 'center',
                          gap: '6px',
                          fontWeight: '500'
                        }">
                          <i class="bi bi-info-circle"></i>
                          Key Concept Understanding
                        </span>
                      </div>
                    </div>
                  </div> <!-- End premium-question-card -->
                </div> <!-- End questions-wrapper -->
              </div>
              
              <div class="case-navigation-buttons">
                <button 
                  class="btn btn-secondary me-2" 
                  @click="previousCase"
                  :disabled="currentCaseIndex === 0"
                >
                  <i class="bi bi-arrow-left"></i> Previous Case
                </button>
                <button 
                  v-if="currentCaseIndex < diagnosticCases.length - 1"
                  class="btn btn-primary" 
                  @click="nextCase"
                >
                  Next Case <i class="bi bi-arrow-right"></i>
                </button>
                <a 
                  v-if="currentCaseIndex >= diagnosticCases.length - 1"
                  :href="getBackLink()" 
                  class="btn btn-success"
                >
                  Return to Dashboard <i class="bi bi-house"></i>
                </a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Face Verification Modal -->
    <ExamAccessVerification
      :show="showFaceVerification"
      :exam-id="examId"
      :exam-title="exam ? exam.title : ''"
      exam-type="diagnostic"
      :verification-required="true"
      @verified="handleVerificationSuccess"
      @close="showFaceVerification = false"
    />
  </div>
</template>

<script>
import api from '@/services/api';
import ExamAccessVerification from '@/components/ExamAccessVerification.vue';

export default {
  name: 'DiagnosticExam',
  components: {
    ExamAccessVerification
  },
  props: {
    examId: {
      type: [String, Number],
      required: true
    },
    examType: {
      type: String,
      default: 'DIAGNOSTIC',
      validator: (value) => value === 'DIAGNOSTIC' || value === 'diagnostic'
    },
    forceType: {
      type: String,
      default: 'DIAGNOSTIC'
    }
  },
  data() {
    return {
      exam: null,
      diagnosticCases: [],
      currentCaseIndex: 0,
      currentCase: null,
      currentCaseQuestions: [],
      userCaseAnswers: {},  // Stores answers by question ID
      examStarted: false,
      examCompleted: false,
      examLoading: true,
      remainingTime: 0,
      timer: null,
      attemptId: null,
      diagnosticResults: null,
      retryAttempted: false,
      lightboxActive: false,
      lightboxImage: '',
      lightboxIndex: 0,
      lightboxImages: [],
      // Face verification
      showFaceVerification: false,
      faceVerified: false
    };
  },
  computed: {
    answeredQuestionsCount() {
      return Object.keys(this.userCaseAnswers).length;
    },
    totalQuestionsCount() {
      return this.diagnosticCases.reduce((total, caseItem) => {
        return total + (caseItem.questions?.length || 0);
      }, 0);
    },
    previousAttempts() {
      // Try to get attempts from local storage
      const storedAttempts = localStorage.getItem(`exam_${this.examId}_attempts`);
      if (storedAttempts) {
        try {
          const attempts = JSON.parse(storedAttempts);
          if (Array.isArray(attempts) && attempts.length > 0) {
            // Format data for display
            return attempts.map(attempt => ({
              id: attempt.id,
              date: new Date(attempt.completed_at || attempt.started_at).toLocaleDateString(),
              score: attempt.score || 0,
              completed: !!attempt.completed_at
            }));
          }
        } catch (e) {
          console.error('Error parsing stored attempts:', e);
        }
      }
      
      // Fallback to empty array if no attempts found
      return [];
    },
    bestScore() {
      if (this.previousAttempts.length === 0) return 0;
      return Math.max(...this.previousAttempts.map(attempt => attempt.score));
    }
  },
  async mounted() {
    console.log('DiagnosticExam component mounted with examId:', this.examId);
    
    // Add event listener for Escape key to close lightbox
    window.addEventListener('keydown', this.handleKeydown);
    
    // Load exam data
    await this.fetchExamData();
  },
  
  beforeUnmount() {
    // Clean up event listeners
    window.removeEventListener('keydown', this.handleKeydown);
    
    // Ensure body scrolling is enabled when component is destroyed
    document.body.style.overflow = 'auto';
    
    // Clean up timer
    clearInterval(this.timer);
  },
  methods: {
    getBackLink() {
      return '/dashboard';
    },
    async fetchExamData() {
      try {
        console.log('Fetching DIAGNOSTIC exam data with ID:', this.examId);
        const response = await api.getDiagnosticExam(this.examId);
        
        if (response && response.data) {
          console.log('Received exam data:', response.data);
          
          // Process the exam data
          this.exam = {
            id: response.data.id,
            title: response.data.title,
            type: 'DIAGNOSTIC',
            description: response.data.description,
            duration_minutes: response.data.duration_minutes,
            duration: `${response.data.duration_minutes} minutes`,
            passmark: response.data.passmark || 70,
            case_count: response.data.case_count
          };
          
          // Set the exam timer
          this.remainingTime = this.exam.duration_minutes * 60; // Convert minutes to seconds
          
          // Fetch diagnostic cases
          await this.fetchDiagnosticCases(this.examId);
          
        } else {
          console.error('Invalid or empty response when fetching exam data');
          alert('Failed to load exam data. Please try refreshing the page.');
        }
      } catch (error) {
        console.error('Error fetching exam data:', error);
        
        // Show detailed error information
        if (error.response) {
          console.error('Server response:', {
            status: error.response.status,
            data: error.response.data
          });
        }
        
        alert('There was an error loading the exam. Please try again later.');
      } finally {
        this.examLoading = false;
      }
    },
    async fetchDiagnosticCases(examId) {
      try {
        console.log('Fetching diagnostic cases for exam ID:', examId);
        const response = await api.getDiagnosticCases(examId);
        
        if (response && response.data) {
          this.diagnosticCases = response.data;
          console.log(`Loaded ${this.diagnosticCases.length} diagnostic cases`);
          
          // Set the first case as current
          if (this.diagnosticCases.length > 0) {
            this.setCurrentCase(0);
          }
        } else {
          console.error('Invalid or empty response when fetching diagnostic cases');
        }
      } catch (error) {
        console.error('Error fetching diagnostic cases:', error);
        
        // Show detailed error information
        if (error.response) {
          console.error('Server response:', {
            status: error.response.status,
            data: error.response.data
          });
        }
        
        alert('There was an error loading diagnostic cases. Please try again later.');
      }
    },
    setCurrentCase(index) {
      if (index >= 0 && index < this.diagnosticCases.length) {
        this.currentCaseIndex = index;
        this.currentCase = this.diagnosticCases[index];
        this.currentCaseQuestions = this.currentCase.questions || [];
        console.log('Set current case:', this.currentCase);
        
        // Debug logging for missing fields
        console.log('Case Details:');
        console.log('- Title:', this.currentCase.title);
        console.log('- Instructions:', this.currentCase.instructions || 'NOT PROVIDED');
        console.log('- Clinical Context:', this.currentCase.clinical_context || 'NOT PROVIDED');
        console.log('- Presenting Symptoms:', this.currentCase.presenting_symptoms || 'NOT PROVIDED');
        console.log('- Main Image Raw:', this.currentCase.image || 'NOT PROVIDED');
        console.log('- Main Image URL:', this.currentCase.image_url || 'NOT PROVIDED');
        console.log('- getMainImageUrl():', this.getMainImageUrl() || 'NOT PROVIDED');
        console.log('- Additional Images:', this.currentCase.additional_images || 'NOT PROVIDED');
        console.log('- Additional Image URLs:', this.currentCase.additional_image_urls || 'NOT PROVIDED');
        console.log('- Questions:', this.currentCaseQuestions.length);
      }
    },
    nextCase() {
      if (this.currentCaseIndex < this.diagnosticCases.length - 1) {
        this.setCurrentCase(this.currentCaseIndex + 1);
      }
    },
    previousCase() {
      if (this.currentCaseIndex > 0) {
        this.setCurrentCase(this.currentCaseIndex - 1);
      }
    },
    goToCase(index) {
      this.setCurrentCase(index);
    },
    setAnswer(questionId, answerValue) {
      console.log(`Setting answer for question ${questionId} to:`, answerValue);
      this.userCaseAnswers[questionId] = answerValue;
    },
    caseAnswered(caseIndex) {
      // Check if we have answers for at least one question in this case
      const caseItem = this.diagnosticCases[caseIndex];
      if (!caseItem || !caseItem.questions || caseItem.questions.length === 0) {
        return false;
      }
      
      return caseItem.questions.some(q => this.userCaseAnswers[q.id] !== undefined);
    },
    beginExam() {
      // Show face verification first
      this.showFaceVerification = true;
    },
    
    handleVerificationSuccess(data) {
      console.log('Face verification successful:', data);
      
      if (data.success) {
        this.showFaceVerification = false;
        this.faceVerified = true;
        // Actually start the exam after verification
        this.startDiagnosticExam();
      } else {
        alert(`Face verification failed. Confidence: ${data.confidence.toFixed(0)}%. Please ensure good lighting and try again.`);
      }
    },
    
    startDiagnosticExam() {
      console.log('beginExam called - starting diagnostic exam');
      console.log('Exam details:', { 
        id: this.examId, 
        type: this.exam?.type,
        duration: this.exam?.duration,
        title: this.exam?.title
      });
      
      // Start the exam
      this.examStarted = true;
      
      // Start the timer
      this.startTimer();
      
      // Attempt to immediately create the exam attempt in the database
      console.log('Diagnostic exam detected - creating attempt immediately');
      this.saveExamAttempt();
      
      // Notify the admin (in a real implementation)
      this.notifyAdmin();
    },
    startTimer() {
      this.timer = setInterval(() => {
        if (this.remainingTime > 0) {
          this.remainingTime--;
        } else {
          // Time's up - end the exam
          this.endExam();
        }
      }, 1000);
    },
    notifyAdmin() {
      console.log('Notifying admin of exam access - would send email in production');
    },
    formatTime(seconds) {
      const hours = Math.floor(seconds / 3600);
      const minutes = Math.floor((seconds % 3600) / 60);
      const secs = seconds % 60;
      
      return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    },
    async finishDiagnosticExam() {
      if (confirm('Are you sure you want to finish the exam? You will not be able to change your answers after this.')) {
        await this.endExam();
      }
    },
    async endExam() {
      // Stop the timer
      clearInterval(this.timer);
      
      // Mark the exam as completed
      this.examCompleted = true;
      
      // Calculate results
      this.prepareResults();
      
      // Save the exam attempt to the backend
      this.saveExamAttempt();
    },
    prepareResults() {
      console.log('Preparing diagnostic exam results');
      
      // Initialize results structure
      const results = {
        totalQuestions: 0,
        correctCount: 0,
        totalScore: 0,
        completedAt: new Date().toISOString(),
        caseResults: []
      };
      
      // Process each case
      this.diagnosticCases.forEach((caseItem) => {
        // Initialize case result
        const caseResult = {
          caseId: caseItem.id,
          caseTitle: caseItem.title,
          totalQuestions: caseItem.questions?.length || 0,
          correctCount: 0,
          score: 0
        };
        
        // Count case questions and correct answers
        if (caseItem.questions && caseItem.questions.length > 0) {
          results.totalQuestions += caseItem.questions.length;
          
          // Process each question
          caseItem.questions.forEach(question => {
            const userAnswer = this.userCaseAnswers[question.id];
            
            if (question.type === 'multiple_choice' || question.question_type === 'multiple_choice') {
              const correctOption = question.options.findIndex(opt => opt.is_correct);
              
              if (userAnswer === correctOption) {
                caseResult.correctCount++;
                results.correctCount++;
              }
            } else {
              // Text/short answer questions
              if (this.isTextAnswerCorrect(question, userAnswer)) {
                caseResult.correctCount++;
                results.correctCount++;
              }
            }
          });
          
          // Calculate case score
          if (caseResult.totalQuestions > 0) {
            caseResult.score = Math.round((caseResult.correctCount / caseResult.totalQuestions) * 100);
          }
        }
        
        // Add to results
        results.caseResults.push(caseResult);
      });
      
      // Calculate overall score
      if (results.totalQuestions > 0) {
        results.totalScore = Math.round((results.correctCount / results.totalQuestions) * 100);
      }
      
      // Store results
      this.diagnosticResults = results;
      console.log('Diagnostic exam results:', this.diagnosticResults);
    },
    async saveExamAttempt() {
      try {
        // Get the current user ID
        const user = JSON.parse(localStorage.getItem('user') || '{}');
        if (!user.id) {
          console.warn('User not logged in or ID not available, exam attempt not saved');
          return;
        }
        
        // Calculate time spent for proper database recording
        const timeSpentInSeconds = this.exam.duration_minutes * 60 - this.remainingTime;
        
        // Prepare answers data for submission
        const answers = [];
        
        // Format answers from all cases
        this.diagnosticCases.forEach(caseItem => {
          if (caseItem.questions && caseItem.questions.length > 0) {
            caseItem.questions.forEach(question => {
              const userAnswer = this.userCaseAnswers[question.id];
              
              if (userAnswer !== undefined) {
                // For multiple choice, convert to array format expected by API
                const formattedAnswer = question.type === 'multiple_choice' || question.question_type === 'multiple_choice'
                  ? [userAnswer]  // API expects array for multiple choice
                  : userAnswer;   // Text answers stay as is
                
                answers.push({
                  question_id: question.id,
                  selected_option_ids: Array.isArray(formattedAnswer) 
                    ? formattedAnswer 
                    : [formattedAnswer]  // Ensure array format for API
                });
              }
            });
          }
        });
        
        // Start a new attempt if we don't have an existing attempt ID
        if (!this.attemptId) {
          try {
            // Start a new attempt
            console.log(`Starting diagnostic exam attempt for exam ID:`, this.examId);
            
            // Try the fallback approach in mockExamsApi
            // The startExamAttempt method already has a fallback implementation
            const startResponse = await api.startExamAttempt(this.examId, 'MCQ');
            
            this.attemptId = startResponse.data.id;
            console.log('Started new diagnostic exam attempt in database (direct method):', this.attemptId);
          } catch (error) {
            console.error('Failed to start diagnostic exam attempt in database:', error);
            console.error('Request details:', { examId: this.examId, examType: 'DIAGNOSTIC' });
            
            // Show more detailed error message to user
            alert('There was an issue saving your exam attempt. Please check your internet connection and try again. Error: ' + (error.message || 'Unknown error'));
            return; // Exit early if we can't create the attempt
          }
        }
        
        // Complete the attempt with answers if we have an attempt ID and the exam is completed
        // Only send completion data if the exam is actually completed
        if (this.attemptId && this.examCompleted) {
          try {
            // Add extra metadata to the completeExamAttempt request
            const attemptData = {
              answers: answers,
              meta: {
                time_spent_seconds: timeSpentInSeconds,
                results: this.diagnosticResults,
                completion_time: new Date().toISOString()
              }
            };
            
            // Submit to database using the helper method
            console.log(`Completing diagnostic exam attempt with ID:`, this.attemptId);
            const completeResponse = await api.completeExamAttempt(this.attemptId, attemptData, 'MCQ');
            console.log('Successfully completed diagnostic exam attempt in database (direct method):', completeResponse.data);
            
            // No localStorage storage - we rely entirely on the database
          } catch (error) {
            console.error('Failed to complete diagnostic exam attempt in database:', error);
            // Show error message to user
            alert('There was an issue saving your exam results. Your progress may not be recorded correctly.');
          }
        }
      } catch (error) {
        console.error('Error in saveExamAttempt:', error);
        
        // Enhanced error logging
        if (error.response) {
          console.error('Server response:', {
            status: error.response.status,
            data: error.response.data,
            headers: error.response.headers
          });
        } else if (error.request) {
          console.error('Request made but no response received:', error.request);
        } else {
          console.error('Request setup error:', error.message);
        }
        
        // More informative error message for user
        let errorMsg = 'An unexpected error occurred while saving your exam attempt.';
        
        // Add more details for a better user experience
        if (error.response?.status === 400) {
          errorMsg = 'There was an issue with the exam data. Please try refreshing the page.';
        } else if (error.response?.status === 401) {
          errorMsg = 'Your session has expired. Please log in again to continue.';
        } else if (error.response?.status === 404) {
          errorMsg = 'The exam could not be found. It may have been removed or modified.';
        } else if (error.response?.status >= 500) {
          errorMsg = 'There was a server error. Please try again later or contact support.';
        }
        
        // Show error to user
        alert(errorMsg);
        
        // Try one more time with a different approach if it's a first-time error
        if (!this.retryAttempted && !this.attemptId) {
          console.log('Will attempt one retry for diagnostic exam');
          this.retryAttempted = true;
          
          // Short timeout before retry
          setTimeout(() => {
            console.log('Executing retry attempt');
            this.saveExamAttempt();
          }, 1000);
        }
      }
    },
    getClinicalInfo() {
      if (!this.currentCase) return null;
      
      console.log('Getting clinical info from case:', this.currentCase);
      
      // Primary field is clinical_context from the admin panel
      if (this.currentCase.clinical_context && 
          typeof this.currentCase.clinical_context === 'string' && 
          this.currentCase.clinical_context.trim() !== '') {
        console.log('Using primary clinical_context field from admin panel');
        return this.currentCase.clinical_context;
      }
      
      // Fallback 1: Check for description field
      if (this.currentCase.description && 
          typeof this.currentCase.description === 'string' && 
          this.currentCase.description.trim() !== '') {
        console.log('Using description field as fallback');
        return this.currentCase.description;
      }
      
      // Fallback 2: Check for instructions field
      if (this.currentCase.instructions && 
          typeof this.currentCase.instructions === 'string' && 
          this.currentCase.instructions.trim() !== '') {
        console.log('Using instructions field as fallback');
        return this.currentCase.instructions;
      }
      
      // Last resort: Return a default message
      return 'No clinical information available for this case.';
    },
    getMainImageUrl() {
      if (!this.currentCase) return '';
      
      console.log('Getting main image URL for case:', this.currentCase.id);
      console.log('Available image fields:', {
        image: this.currentCase.image,
        image_url: this.currentCase.image_url
      });
      
      // Prioritize image_url (processed by backend) over raw image field
      let imageUrl = this.currentCase.image_url || this.currentCase.image;
      
      if (!imageUrl) {
        console.log('No image URL found for case');
        return '';
      }
      
      // If image_url is already a full URL, use it directly
      if (this.currentCase.image_url && (this.currentCase.image_url.startsWith('http://') || this.currentCase.image_url.startsWith('https://'))) {
        console.log('Using processed image_url:', this.currentCase.image_url);
        return this.currentCase.image_url;
      }
      
      // Otherwise, process through getFullImageUrl
      console.log('Processing image URL:', imageUrl);
      return this.getFullImageUrl(imageUrl);
    },
    getAdditionalImages() {
      if (!this.currentCase) {
        return [];
      }
      
      console.log('Getting additional images for case:', this.currentCase.id);
      console.log('Additional image fields:', {
        additional_images: this.currentCase.additional_images,
        additional_image_urls: this.currentCase.additional_image_urls
      });
      
      // Prioritize processed URLs from backend
      if (this.currentCase.additional_image_urls && Array.isArray(this.currentCase.additional_image_urls)) {
        console.log('Using processed additional_image_urls:', this.currentCase.additional_image_urls);
        return this.currentCase.additional_image_urls;
      }
      
      // Fallback to processing raw image names
      if (this.currentCase.additional_images && Array.isArray(this.currentCase.additional_images)) {
        console.log('Processing additional_images:', this.currentCase.additional_images);
        return this.currentCase.additional_images.map(img => this.getFullImageUrl(img));
      }
      
      return [];
    },
    getFullImageUrl(imagePath) {
      try {
        // Handle null, undefined, or empty strings
        if (!imagePath) {
          return '';
        }
        
        // Make sure imagePath is a string
        if (typeof imagePath !== 'string') {
          imagePath = String(imagePath);
        }
        
        // Handle absolute URLs (http://, https://, //)
        if (imagePath.startsWith('http://') || imagePath.startsWith('https://') || imagePath.startsWith('//')) {
          return imagePath; // Already absolute
        }
        
        // Get the base URL from the API service but remove /api/ suffix for media files
        const apiBaseUrl = api.getBaseUrl ? api.getBaseUrl() : 'http://tutormed.org/api/';
        // For media files, we need the server base URL without /api/
        const serverBaseUrl = apiBaseUrl.replace('/api/', '').replace('/api', '');
        const cleanBaseUrl = serverBaseUrl.endsWith('/') ? serverBaseUrl.slice(0, -1) : serverBaseUrl;
        
        // For diagnostic images, ensure they're stored locally in media/diagnostic_cases/images/
        if (!imagePath.startsWith('/')) {
          // If it's just a filename, assume it's in diagnostic_cases/images/
          return `${cleanBaseUrl}/media/diagnostic_cases/images/${imagePath}`;
        }
        
        // Handle paths that already include /media/ or /static/
        if (imagePath.startsWith('/media/') || imagePath.startsWith('/static/')) {
          return `${cleanBaseUrl}${imagePath}`;
        }
        
        // Default case - just prepend the base URL
        return `${cleanBaseUrl}${imagePath}`;
      } catch (error) {
        console.error('Error formatting image URL:', error);
        return '';
      }
    },
    handleImageError(event) {
      console.error('Image failed to load:', {
        src: event.target.src,
        case_id: this.currentCase?.id,
        image_field: this.currentCase?.image,
        image_url_field: this.currentCase?.image_url
      });
      
      // Don't show placeholder - hide the image element instead
      event.target.style.display = 'none';
      
      // Check if parent container should be hidden (if all images are hidden)
      this.$nextTick(() => {
        const container = event.target.closest('.main-image-container, .additional-images-container');
        if (container) {
          const visibleImages = container.querySelectorAll('img:not([style*="display: none"])');
          if (visibleImages.length === 0) {
            container.style.display = 'none';
          }
        }
      });
    },
    
    handleImageLoad() {
      // This method exists to track successful image loads
      // It can be expanded later if needed
    },
    
    getPerformanceClass(percentage) {
      if (percentage >= 90) return 'performance-excellent';
      if (percentage >= 80) return 'performance-good';
      if (percentage >= 70) return 'performance-satisfactory';
      if (percentage >= 60) return 'performance-needs-improvement';
      return 'performance-poor';
    },
    
    getPerformanceIcon(percentage) {
      if (percentage >= 90) return 'bi bi-star-fill';
      if (percentage >= 80) return 'bi bi-emoji-smile-fill';
      if (percentage >= 70) return 'bi bi-check-circle-fill';
      if (percentage >= 60) return 'bi bi-exclamation-triangle-fill';
      return 'bi bi-x-circle-fill';
    },
    
    getPerformanceText(percentage) {
      if (percentage >= 90) return 'Outstanding Performance';
      if (percentage >= 80) return 'Good Performance';
      if (percentage >= 70) return 'Satisfactory Performance';
      if (percentage >= 60) return 'Needs Improvement';
      return 'Requires Further Study';
    },
    
    getQuestionTypeClass(question) {
      const type = question.question_type || question.type || 'multiple_choice';
      switch (type) {
        case 'multiple_choice': return 'type-mcq';
        case 'short_answer': return 'type-short';
        case 'long_answer': return 'type-long';
        case 'finding': return 'type-finding';
        default: return 'type-mcq';
      }
    },
    
    getQuestionTypeIcon(question) {
      const type = question.question_type || question.type || 'multiple_choice';
      switch (type) {
        case 'multiple_choice': return 'bi bi-list-ul';
        case 'short_answer': return 'bi bi-chat-left-text';
        case 'long_answer': return 'bi bi-file-text';
        case 'finding': return 'bi bi-search';
        default: return 'bi bi-list-ul';
      }
    },
    
    getQuestionTypeName(question) {
      const type = question.question_type || question.type || 'multiple_choice';
      switch (type) {
        case 'multiple_choice': return 'Multiple Choice';
        case 'short_answer': return 'Short Answer';
        case 'long_answer': return 'Essay Question';
        case 'finding': return 'Clinical Finding';
        default: return 'Multiple Choice';
      }
    },
    showLightbox(imageUrl) {
      // Collect all available images in current case
      this.lightboxImages = [];
      if (this.getMainImageUrl()) {
        this.lightboxImages.push(this.getMainImageUrl());
      }
      this.lightboxImages = [...this.lightboxImages, ...this.getAdditionalImages()];
      
      // Find the index of the selected image
      this.lightboxIndex = this.lightboxImages.findIndex(img => img === imageUrl);
      if (this.lightboxIndex === -1) this.lightboxIndex = 0;
      
      // Show lightbox with the selected image
      this.lightboxImage = imageUrl;
      this.lightboxActive = true;
      
      // Prevent scrolling of the body when lightbox is open
      document.body.style.overflow = 'hidden';
    },
    
    hideLightbox() {
      // Hide the lightbox
      this.lightboxActive = false;
      // Re-enable scrolling
      document.body.style.overflow = 'auto';
    },
    
    prevLightboxImage() {
      if (this.lightboxImages.length <= 1) return;
      this.lightboxIndex = (this.lightboxIndex - 1 + this.lightboxImages.length) % this.lightboxImages.length;
      this.lightboxImage = this.lightboxImages[this.lightboxIndex];
    },
    
    nextLightboxImage() {
      if (this.lightboxImages.length <= 1) return;
      this.lightboxIndex = (this.lightboxIndex + 1) % this.lightboxImages.length;
      this.lightboxImage = this.lightboxImages[this.lightboxIndex];
    },
    
    handleKeydown(event) {
      if (!this.lightboxActive) return;
      
      switch(event.key) {
        case 'Escape':
          this.hideLightbox();
          break;
        case 'ArrowLeft':
          this.prevLightboxImage();
          break;
        case 'ArrowRight':
          this.nextLightboxImage();
          break;
      }
    },

    isTextAnswerCorrect(question, userAnswer) {
      if (!question || !question.model_answer || !userAnswer) {
        return false;
      }
      
      // For text-based answers, do a fuzzy match
      // In a real implementation, you would use NLP or keyword matching
      // This is a simplified example
      const modelAnswer = question.model_answer.toLowerCase().trim();
      const userAnswerText = userAnswer.toLowerCase().trim();
      
      // Check if the user's answer contains keywords from the model answer
      // This is very simplistic - in production you'd want better matching
      const keywords = question.keywords || [];
      
      if (keywords.length > 0) {
        // If we have specific keywords, check for those
        return keywords.some(keyword => 
          userAnswerText.includes(keyword.toLowerCase().trim())
        );
      }
      
      // Simple substring matching as fallback
      return userAnswerText.includes(modelAnswer) || 
             modelAnswer.includes(userAnswerText);
    },
    isQuestionCorrect(question) {
      if (!question || !this.userCaseAnswers[question.id]) {
        return false;
      }
      
      const userAnswer = this.userCaseAnswers[question.id];
      
      // For multiple choice questions
      if (question.type === 'multiple_choice' || question.question_type === 'multiple_choice') {
        const selectedOption = question.options[userAnswer];
        return selectedOption && selectedOption.is_correct;
      }
      
      // For text-based questions
      return this.isTextAnswerCorrect(question, userAnswer);
    },
    getCurrentCaseResult() {
      if (!this.diagnosticResults || !this.currentCase) {
        return null;
      }
      
      return this.diagnosticResults.caseResults.find(
        result => result.caseId === this.currentCase.id
      );
    },
    getScoreClass(score) {
      if (score >= 80) return 'score-high';
      if (score >= 60) return 'score-medium';
      return 'score-low';
    }
  },
  // Using Vue lifecycle hooks for proper cleanup
}
</script>

<style lang="scss">
/* Root variables */
:root {
  --primary-color: #00cec9;
  --secondary-color: #0984e3;
  --light-color: #f8f9fa;
  --gray-color: #6c757d;
}

/* Page layout */
.diagnostic-exam-page {
  background-color: var(--light-color);
  min-height: 100vh;
}

/* General styles for diagnostic exams */
.diagnostic-interface {
  .case-panel {
    background: white;
    border-radius: 10px;
    box-shadow: 0 5px 15px rgba(0,0,0,0.05);
    padding: 25px;
    margin-bottom: 20px;
  }
  
  .case-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-bottom: 15px;
    border-bottom: 1px solid #eee;
    margin-bottom: 20px;
    
    h2, h3 {
      margin: 0;
      color: var(--primary-color);
    }
    
    .case-navigation {
      font-weight: 600;
    }
  }
  
  .clinical-info {
    background-color: #f8f9fa;
    border-radius: 8px;
    border-left: 4px solid #6f42c1;
  }
  
  /* Enhanced image styling */
  .image-container {
    margin: 20px 0;
  }
  
  .main-image-container {
    .image-card {
      border-radius: 8px;
      overflow: hidden;
      box-shadow: 0 2px 12px rgba(0,0,0,0.08);
      
      .image-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 12px 15px;
        background: linear-gradient(to right, #f8f9fa, #e9ecef);
        border-bottom: 1px solid #e9ecef;
        
        h5 {
          margin: 0;
          font-size: 1rem;
          color: #495057;
          
          i {
            color: var(--primary-color);
          }
        }
        
        .btn-zoom {
          background: none;
          border: none;
          color: #6c757d;
          cursor: pointer;
          transition: all 0.2s ease;
          padding: 5px 8px;
          border-radius: 4px;
          
          &:hover {
            background-color: rgba(0,0,0,0.05);
            color: var(--primary-color);
          }
        }
      }
      
      .image-wrapper {
        position: relative;
        background-color: #f8f9fa;
        text-align: center;
        overflow: hidden;
        
        .main-image {
          max-height: 500px;
          max-width: 100%;
          display: block;
          margin: 0 auto;
          transition: transform 0.3s ease;
        }
        
        .image-overlay {
          position: absolute;
          top: 0;
          left: 0;
          right: 0;
          bottom: 0;
          background: rgba(0,0,0,0.5);
          display: flex;
          justify-content: center;
          align-items: center;
          opacity: 0;
          transition: opacity 0.3s ease;
          cursor: pointer;
          
          span {
            color: white;
            font-weight: 500;
            background: rgba(0,0,0,0.5);
            padding: 8px 15px;
            border-radius: 4px;
            
            i {
              margin-right: 5px;
            }
          }
        }
        
        &:hover {
          .image-overlay {
            opacity: 1;
          }
        }
      }
    }
  }
  
  .additional-images-container {
    h5 {
      color: #495057;
      font-size: 1rem;
      
      i {
        color: var(--primary-color);
      }
    }
    
    .image-gallery {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
      gap: 15px;
      
      .gallery-item {
        .thumbnail-wrapper {
          position: relative;
          border-radius: 6px;
          overflow: hidden;
          box-shadow: 0 2px 8px rgba(0,0,0,0.1);
          aspect-ratio: 4/3;
          
          .thumbnail {
            width: 100%;
            height: 100%;
            object-fit: cover;
            transition: transform 0.3s ease;
          }
          
          .thumbnail-overlay {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0,0,0,0.5);
            display: flex;
            justify-content: center;
            align-items: center;
            opacity: 0;
            transition: opacity 0.3s ease;
            cursor: pointer;
            
            i {
              color: white;
              font-size: 1.5rem;
            }
          }
          
          &:hover {
            .thumbnail {
              transform: scale(1.05);
            }
            
            .thumbnail-overlay {
              opacity: 1;
            }
          }
        }
      }
    }
  }
  
  /* Lightbox styles */
  .lightbox-modal {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0,0,0,0.9);
    z-index: 1050;
    display: flex;
    justify-content: center;
    align-items: center;
    
    .lightbox-content {
      position: relative;
      max-width: 90vw;
      max-height: 90vh;
      
      .lightbox-close {
        position: absolute;
        top: -40px;
        right: 0;
        background: none;
        border: none;
        color: white;
        font-size: 1.25rem;
        cursor: pointer;
        z-index: 1060;
        padding: 5px;
        
        &:hover {
          color: var(--primary-color);
        }
      }
      
      .lightbox-counter {
        position: absolute;
        bottom: -30px;
        left: 0;
        right: 0;
        text-align: center;
        color: white;
        font-size: 0.9rem;
      }
      
      .lightbox-nav {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        background: rgba(255,255,255,0.2);
        border: none;
        color: white;
        font-size: 1.5rem;
        cursor: pointer;
        z-index: 1060;
        width: 40px;
        height: 40px;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        transition: all 0.2s ease;
        
        &:hover {
          background: rgba(255,255,255,0.3);
        }
        
        &.lightbox-prev {
          left: -60px;
        }
        
        &.lightbox-next {
          right: -60px;
        }
        
        /* Responsive adjustments for smaller screens */
        @media (max-width: 768px) {
          width: 36px;
          height: 36px;
          font-size: 1.2rem;
          
          &.lightbox-prev {
            left: -45px;
          }
          
          &.lightbox-next {
            right: -45px;
          }
        }
      }
      
      .lightbox-image-container {
        display: flex;
        justify-content: center;
        align-items: center;
        
        .lightbox-image {
          max-width: 100%;
          max-height: 90vh;
          object-fit: contain;
          border-radius: 4px;
          transition: transform 0.3s ease;
        }
      }
    }
  }
  
  .diagnostic-questions,
  .diagnostic-questions-review {
    margin: 30px 0;
    
    h3 {
      color: var(--primary-color);
      margin-bottom: 20px;
    }
    
    .question-item {
      background-color: #f8f9fa;
      border-radius: 8px;
      
      .question-text {
        font-size: 1.1rem;
      }
      
      .form-check {
        margin-left: 10px;
      }
      
      /* Enhanced Option Cards */
      .options-container {
        .option-card {
          background: white;
          border: 2px solid #e9ecef;
          border-radius: 12px;
          padding: 0;
          transition: all 0.3s ease;
          overflow: hidden;
          
          &.option-correct {
            border-color: #28a745;
            background: linear-gradient(135deg, #d4edda 0%, #f8fff9 100%);
            box-shadow: 0 4px 15px rgba(40, 167, 69, 0.2);
          }
          
          &.option-incorrect {
            border-color: #dc3545;
            background: linear-gradient(135deg, #f8d7da 0%, #fff5f5 100%);
            box-shadow: 0 4px 15px rgba(220, 53, 69, 0.2);
          }
          
          &.option-neutral {
            border-color: #6c757d;
            background: #f8f9fa;
          }
          
          .option-content {
            padding: 20px;
            display: flex;
            align-items: flex-start;
            gap: 15px;
            
            .option-indicator {
              flex-shrink: 0;
              
              .indicator {
                width: 40px;
                height: 40px;
                border-radius: 50%;
                display: flex;
                align-items: center;
                justify-content: center;
                font-weight: 600;
                font-size: 1.1rem;
                
                &.correct {
                  background: #28a745;
                  color: white;
                  font-size: 1.2rem;
                }
                
                &.incorrect {
                  background: #dc3545;
                  color: white;
                  font-size: 1.2rem;
                }
                
                &.selected {
                  background: #007bff;
                  color: white;
                  font-size: 1.2rem;
                }
                
                &.letter {
                  background: #f8f9fa;
                  border: 2px solid #dee2e6;
                  color: #495057;
                }
              }
            }
            
            .option-text {
              flex: 1;
              font-size: 1.05rem;
              line-height: 1.6;
              color: #333;
            }
            
            .option-status {
              .status-badge {
                padding: 6px 12px;
                border-radius: 20px;
                font-size: 0.85rem;
                font-weight: 600;
                
                &.correct {
                  background: #28a745;
                  color: white;
                }
                
                &.incorrect {
                  background: #dc3545;
                  color: white;
                }
              }
            }
          }
        }
      }
      
      /* Enhanced Answer Review Section */
      .answer-review-section {
        .answer-card {
          background: white;
          border-radius: 12px;
          margin-bottom: 20px;
          overflow: hidden;
          box-shadow: 0 4px 12px rgba(0,0,0,0.08);
          
          &.user-answer {
            border-left: 4px solid #007bff;
          }
          
          &.model-answer {
            border-left: 4px solid #28a745;
          }
          
          .answer-header {
            background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
            padding: 15px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            
            .answer-title {
              font-weight: 600;
              color: #495057;
              display: flex;
              align-items: center;
              gap: 8px;
              
              i {
                font-size: 1.1rem;
              }
            }
            
            .answer-score {
              .score-badge {
                padding: 6px 12px;
                border-radius: 20px;
                font-size: 0.85rem;
                font-weight: 600;
                display: flex;
                align-items: center;
                gap: 5px;
                
                &.excellent {
                  background: #28a745;
                  color: white;
                }
                
                &.needs-improvement {
                  background: #ffc107;
                  color: #212529;
                }
              }
            }
          }
          
          .answer-content {
            padding: 20px;
            font-size: 1.05rem;
            line-height: 1.6;
            color: #333;
          }
        }
      }
      
      /* Enhanced Explanation Card */
      .explanation-card {
        background: linear-gradient(135deg, #fff3cd 0%, #fff8e1 100%);
        border: 2px solid #ffc107;
        border-radius: 12px;
        margin-top: 25px;
        overflow: hidden;
        box-shadow: 0 4px 15px rgba(255, 193, 7, 0.2);
        
        .explanation-header {
          background: linear-gradient(135deg, #ffc107 0%, #ffca2c 100%);
          padding: 15px 20px;
          display: flex;
          align-items: center;
          gap: 10px;
          
          i {
            font-size: 1.3rem;
            color: #212529;
          }
          
          span {
            font-weight: 600;
            font-size: 1.1rem;
            color: #212529;
          }
        }
        
        .explanation-content {
          padding: 20px;
          font-size: 1.05rem;
          line-height: 1.7;
          color: #333;
          
          p {
            margin-bottom: 15px;
            
            &:last-child {
              margin-bottom: 0;
            }
          }
          
          ul, ol {
            padding-left: 20px;
            margin-bottom: 15px;
            
            li {
              margin-bottom: 8px;
            }
          }
          
          strong {
            color: #495057;
          }
        }
      }
    }
  }
  
  /* Premium Modern Question Review Section */
  .exam-results .results-card .diagnostic-questions-review,
  .diagnostic-questions-review {
    background: linear-gradient(180deg, #ffffff 0%, #f8fafb 100%) !important;
    border-radius: 24px !important;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.08) !important;
    overflow: visible !important;
    margin-top: 40px !important;
    position: relative !important;
    
    &::before {
      content: '';
      position: absolute;
      top: -2px;
      left: -2px;
      right: -2px;
      bottom: -2px;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #10b981 100%);
      border-radius: 24px;
      opacity: 0.1;
      z-index: -1;
    }
    
    .premium-section-header {
      position: relative !important;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
      padding: 40px !important;
      overflow: hidden !important;
      display: block !important;
      
      .header-gradient-overlay {
        position: absolute;
        inset: 0;
        background: radial-gradient(circle at top right, rgba(16, 185, 129, 0.2) 0%, transparent 50%),
                    radial-gradient(circle at bottom left, rgba(139, 92, 246, 0.2) 0%, transparent 50%);
        pointer-events: none;
      }
      
      &::after {
        content: '';
        position: absolute;
        bottom: -50%;
        left: -10%;
        width: 120%;
        height: 100%;
        background: white;
        border-radius: 50% 50% 0 0;
        opacity: 0.1;
      }
      
      .header-content {
        position: relative;
        z-index: 1;
        display: flex;
        justify-content: space-between;
        align-items: center;
        gap: 30px;
        
        .section-title-group {
          display: flex !important;
          align-items: center !important;
          gap: 24px !important;
          
          .icon-wrapper {
            position: relative !important;
            width: 64px !important;
            height: 64px !important;
            
            .icon-glow {
              position: absolute;
              inset: -8px;
              background: radial-gradient(circle, rgba(255, 255, 255, 0.3) 0%, transparent 70%);
              border-radius: 50%;
              animation: subtle-pulse 3s ease-in-out infinite;
            }
            
            @keyframes subtle-pulse {
              0%, 100% { opacity: 0.5; transform: scale(1); }
              50% { opacity: 0.8; transform: scale(1.1); }
            }
            
            i {
              width: 64px !important;
              height: 64px !important;
              background: rgba(255, 255, 255, 0.2) !important;
              backdrop-filter: blur(10px);
              border: 1px solid rgba(255, 255, 255, 0.3) !important;
              border-radius: 16px !important;
              display: flex !important;
              align-items: center !important;
              justify-content: center !important;
              font-size: 28px !important;
              color: white !important;
              box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1) !important;
            }
          }
          
          .title-text {
            .main-title {
              font-size: 2rem !important;
              font-weight: 800 !important;
              color: white !important;
              margin: 0 0 4px 0 !important;
              letter-spacing: -0.02em !important;
              display: block !important;
            }
            
            .subtitle {
              font-size: 1rem !important;
              color: rgba(255, 255, 255, 0.8) !important;
              margin: 0 !important;
              font-weight: 400 !important;
              display: block !important;
            }
          }
        }
        
        .section-stats {
          &.glass-card {
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            border-radius: 16px;
            padding: 16px 24px;
            display: flex;
            align-items: center;
            gap: 20px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
            
            .stat-item {
              text-align: center;
              
              .stat-value {
                display: block;
                font-size: 1.75rem;
                font-weight: 800;
                color: white;
                line-height: 1;
                margin-bottom: 4px;
              }
              
              .stat-label {
                font-size: 0.875rem;
                color: rgba(255, 255, 255, 0.8);
                font-weight: 500;
              }
            }
            
            .stat-divider {
              width: 1px;
              height: 40px;
              background: rgba(255, 255, 255, 0.2);
            }
          }
        }
      }
    }
    
    /* Premium Questions Wrapper */
    .questions-wrapper {
      padding: 40px !important;
      background: transparent !important;
      
      .premium-question-card {
        background: white !important;
        border-radius: 20px !important;
        margin-bottom: 32px !important;
        box-shadow: 0 10px 40px rgba(0, 0, 0, 0.06) !important;
        border: 1px solid rgba(0, 0, 0, 0.04) !important;
        overflow: hidden !important;
        transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1) !important;
        position: relative !important;
        
        &::before {
          content: '';
          position: absolute;
          top: 0;
          left: 0;
          width: 4px;
          height: 100%;
          background: linear-gradient(180deg, #667eea 0%, #764ba2 100%);
          opacity: 0;
          transition: opacity 0.3s ease;
        }
        
        &:hover {
          transform: translateY(-4px);
          box-shadow: 0 20px 60px rgba(0, 0, 0, 0.1);
          
          &::before {
            opacity: 1;
          }
        }
      
        /* Modern Question Header */
        .question-header-modern {
          padding: 28px 32px;
          background: linear-gradient(180deg, #fafbfc 0%, #ffffff 100%);
          border-bottom: 1px solid #e5e7eb;
          display: flex;
          align-items: center;
          gap: 24px;
          
          .question-number-section {
            display: flex;
            align-items: center;
            gap: 16px;
            
            .number-circle {
              position: relative;
              width: 56px;
              height: 56px;
              background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
              border-radius: 16px;
              display: flex;
              align-items: center;
              justify-content: center;
              box-shadow: 0 8px 24px rgba(102, 126, 234, 0.25);
              
              .number {
                font-size: 1.5rem;
                font-weight: 800;
                color: white;
                position: relative;
                z-index: 1;
              }
              
              .number-glow-effect {
                position: absolute;
                inset: -4px;
                background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                border-radius: 18px;
                opacity: 0.3;
                filter: blur(8px);
                animation: glow-animation 2s ease-in-out infinite alternate;
              }
              
              @keyframes glow-animation {
                from { opacity: 0.3; }
                to { opacity: 0.5; }
              }
            }
            
            .question-line {
              width: 60px;
              height: 2px;
              background: linear-gradient(90deg, #e5e7eb 0%, transparent 100%);
            }
          }
          
          .question-info-section {
            flex: 1;
            display: flex;
            justify-content: space-between;
            align-items: center;
            
            .question-badges {
              display: flex;
              gap: 12px;
              
              .type-badge-modern,
              .points-badge-modern {
                padding: 8px 16px;
                border-radius: 12px;
                font-size: 0.875rem;
                font-weight: 600;
                display: flex;
                align-items: center;
                gap: 8px;
                transition: all 0.2s ease;
                
                i {
                  font-size: 1rem;
                }
                
                span {
                  line-height: 1;
                }
              }
              
              .type-badge-modern {
                &.type-mcq {
                  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
                  color: white;
                  box-shadow: 0 4px 12px rgba(16, 185, 129, 0.2);
                }
                
                &.type-short {
                  background: linear-gradient(135deg, #06b6d4 0%, #0891b2 100%);
                  color: white;
                  box-shadow: 0 4px 12px rgba(6, 182, 212, 0.2);
                }
                
                &.type-long {
                  background: linear-gradient(135deg, #8b5cf6 0%, #7c3aed 100%);
                  color: white;
                  box-shadow: 0 4px 12px rgba(139, 92, 246, 0.2);
                }
                
                &:hover {
                  transform: translateY(-2px);
                  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
                }
              }
              
              .points-badge-modern {
                background: linear-gradient(135deg, #fbbf24 0%, #f59e0b 100%);
                color: #78350f;
                box-shadow: 0 4px 12px rgba(251, 191, 36, 0.2);
                
                i {
                  color: #92400e;
                }
              }
            }
            
            .result-indicator {
              padding: 10px 20px;
              border-radius: 24px;
              font-weight: 600;
              font-size: 0.9rem;
              display: flex;
              align-items: center;
              gap: 8px;
              
              &.correct {
                background: linear-gradient(135deg, #d1fae5 0%, #a7f3d0 100%);
                color: #065f46;
                border: 1px solid #10b981;
                
                i {
                  color: #10b981;
                }
              }
              
              &.incorrect {
                background: linear-gradient(135deg, #fee2e2 0%, #fecaca 100%);
                color: #991b1b;
                border: 1px solid #ef4444;
                
                i {
                  color: #ef4444;
                }
              }
            }
          }
        }
        
        /* Modern Question Content */
        .question-content-modern {
          padding: 32px;
          
          .question-text {
            font-size: 1.125rem;
            line-height: 1.8;
            color: #1f2937;
            font-weight: 500;
            letter-spacing: -0.01em;
          }
        }
        
        /* Modern Options Container */
        .options-modern-container {
          padding: 0 32px 32px;
          
          .option-modern-card {
            background: #fafbfc;
            border: 2px solid #e5e7eb;
            border-radius: 16px;
            padding: 20px 24px;
            margin-bottom: 16px;
            transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
            cursor: pointer;
            position: relative;
            overflow: hidden;
            
            &::before {
              content: '';
              position: absolute;
              top: 0;
              left: 0;
              width: 100%;
              height: 100%;
              background: linear-gradient(135deg, transparent 0%, rgba(102, 126, 234, 0.05) 100%);
              opacity: 0;
              transition: opacity 0.3s ease;
            }
            
            &:hover:not(.is-correct):not(.is-incorrect) {
              border-color: #c7d2fe;
              transform: translateX(4px);
              box-shadow: 0 4px 16px rgba(0, 0, 0, 0.05);
              
              &::before {
                opacity: 1;
              }
            }
            
            &.is-correct {
              background: linear-gradient(135deg, #f0fdf4 0%, #dcfce7 100%);
              border-color: #10b981;
              box-shadow: 0 4px 16px rgba(16, 185, 129, 0.15);
            }
            
            &.is-incorrect {
              background: linear-gradient(135deg, #fef2f2 0%, #fee2e2 100%);
              border-color: #ef4444;
              box-shadow: 0 4px 16px rgba(239, 68, 68, 0.15);
            }
            
            &.is-neutral {
              opacity: 0.6;
              
              &:hover {
                opacity: 0.8;
              }
            }
            
            .option-modern-content {
              display: flex;
              align-items: center;
              gap: 20px;
              
              .option-indicator-modern {
                flex-shrink: 0;
                
                .indicator-icon {
                  width: 44px;
                  height: 44px;
                  border-radius: 12px;
                  display: flex;
                  align-items: center;
                  justify-content: center;
                  font-weight: 700;
                  font-size: 1.25rem;
                  
                  &.success {
                    background: linear-gradient(135deg, #10b981 0%, #059669 100%);
                    color: white;
                    box-shadow: 0 4px 12px rgba(16, 185, 129, 0.3);
                  }
                  
                  &.error {
                    background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
                    color: white;
                    box-shadow: 0 4px 12px rgba(239, 68, 68, 0.3);
                  }
                }
                
                .indicator-letter {
                  width: 44px;
                  height: 44px;
                  background: linear-gradient(135deg, #f3f4f6 0%, #e5e7eb 100%);
                  border: 2px solid #d1d5db;
                  border-radius: 12px;
                  display: flex;
                  align-items: center;
                  justify-content: center;
                  font-weight: 700;
                  font-size: 1.125rem;
                  color: #6b7280;
                  transition: all 0.2s ease;
                }
              }
              
              .option-text-wrapper {
                flex: 1;
                
                .option-text {
                  font-size: 1rem;
                  line-height: 1.6;
                  color: #374151;
                  margin: 0;
                  font-weight: 500;
                }
                
                .option-feedback {
                  margin-top: 8px;
                  
                  .feedback-badge {
                    display: inline-flex;
                    align-items: center;
                    gap: 6px;
                    padding: 4px 12px;
                    border-radius: 20px;
                    font-size: 0.75rem;
                    font-weight: 600;
                    
                    &.success {
                      background: #10b981;
                      color: white;
                    }
                    
                    &.info {
                      background: #3b82f6;
                      color: white;
                    }
                    
                    i {
                      font-size: 0.875rem;
                    }
                  }
                }
              }
            }
          }
        }
        
      
        /* Premium Explanation Card */
        .explanation-premium-card {
          margin: 32px;
          background: linear-gradient(135deg, #faf5ff 0%, #f3e8ff 100%);
          border: 2px solid #e9d5ff;
          border-radius: 20px;
          overflow: hidden;
          box-shadow: 0 10px 30px rgba(139, 92, 246, 0.08);
          position: relative;
          
          &::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 4px;
            background: linear-gradient(90deg, #8b5cf6 0%, #7c3aed 50%, #6366f1 100%);
          }
          
          .explanation-header-modern {
            padding: 24px 28px;
            background: linear-gradient(135deg, rgba(139, 92, 246, 0.05) 0%, rgba(124, 58, 237, 0.05) 100%);
            border-bottom: 1px solid #e9d5ff;
            display: flex;
            align-items: center;
            gap: 16px;
            
            .explanation-icon-wrapper {
              position: relative;
              width: 48px;
              height: 48px;
              
              i {
                width: 48px;
                height: 48px;
                background: linear-gradient(135deg, #8b5cf6 0%, #7c3aed 100%);
                color: white;
                border-radius: 14px;
                display: flex;
                align-items: center;
                justify-content: center;
                font-size: 24px;
                box-shadow: 0 6px 20px rgba(139, 92, 246, 0.3);
              }
              
              .icon-pulse {
                position: absolute;
                inset: -6px;
                background: radial-gradient(circle, rgba(139, 92, 246, 0.3) 0%, transparent 70%);
                border-radius: 16px;
                animation: pulse-light 2s ease-in-out infinite;
              }
              
              @keyframes pulse-light {
                0%, 100% { opacity: 0.3; transform: scale(1); }
                50% { opacity: 0.6; transform: scale(1.05); }
              }
            }
            
            .explanation-title {
              font-size: 1.25rem;
              font-weight: 700;
              color: #6b21a8;
              margin: 0;
              letter-spacing: -0.02em;
            }
          }
          
          .explanation-content-modern {
            padding: 28px;
            
            .explanation-text {
              font-size: 1.05rem;
              line-height: 1.8;
              color: #4b5563;
              
              p {
                margin-bottom: 16px;
                
                &:last-child {
                  margin-bottom: 0;
                }
              }
              
              ul, ol {
                margin-left: 24px;
                margin-bottom: 16px;
                
                li {
                  margin-bottom: 8px;
                  color: #6b7280;
                }
              }
              
              strong {
                color: #7c3aed;
                font-weight: 600;
              }
              
              em {
                color: #8b5cf6;
                font-style: italic;
              }
            }
          }
          
          .explanation-footer {
            padding: 16px 28px;
            background: rgba(139, 92, 246, 0.05);
            border-top: 1px solid #e9d5ff;
            
            .tip-label {
              display: flex;
              align-items: center;
              gap: 8px;
              font-size: 0.875rem;
              color: #7c3aed;
              font-weight: 600;
              
              i {
                font-size: 1rem;
              }
            }
          }
        }
      }
    }
  }
  
  .case-navigation-buttons {
    margin-top: 30px;
    display: flex;
    align-items: center;
  }
  
  .navigation-panel {
    background: white;
    border-radius: 10px;
    box-shadow: 0 5px 15px rgba(0,0,0,0.05);
    padding: 20px;
    position: sticky;
    top: 20px;
    
    .navigation-header {
      padding-bottom: 15px;
      border-bottom: 1px solid #eee;
      margin-bottom: 15px;
      
      h3 {
        font-size: 1.2rem;
        margin-bottom: 0;
        color: var(--primary-color);
      }
    }
    
    .navigation-status {
      display: flex;
      justify-content: space-around;
      margin-bottom: 20px;
      
      .status-item {
        text-align: center;
        
        .status-value {
          font-size: 1.5rem;
          font-weight: 700;
          color: var(--primary-color);
        }
        
        .status-label {
          font-size: 0.9rem;
          color: var(--gray-color);
        }
      }
    }
    
    .case-list {
      h4 {
        color: var(--primary-color);
      }
      
      .list-group-item {
        padding: 12px 15px;
        transition: all 0.2s ease;
        
        &.active {
          background-color: var(--primary-color);
          border-color: var(--primary-color);
        }
      }
    }
  }
}

/* Results styles */
.exam-results {
  padding: 40px 0;
  
  .results-card {
    background: white;
    border-radius: 10px;
    box-shadow: 0 5px 15px rgba(0,0,0,0.05);
    padding: 30px;
  }
  
  .score-display {
    display: flex;
    flex-direction: column;
    align-items: center;
    
    .score-circle {
      width: 100px;
      height: 100px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-bottom: 10px;
      
      &.score-high {
        background-color: rgba(40, 167, 69, 0.2);
        border: 4px solid #28a745;
        color: #28a745;
      }
      
      &.score-medium {
        background-color: rgba(255, 193, 7, 0.2);
        border: 4px solid #ffc107;
        color: #856404;
      }
      
      &.score-low {
        background-color: rgba(220, 53, 69, 0.2);
        border: 4px solid #dc3545;
        color: #dc3545;
      }
      
      .score-value {
        font-size: 1.8rem;
        font-weight: 700;
      }
    }
    
    .score-label {
      font-weight: 600;
      color: #6c757d;
    }
  }
}

@keyframes spinner {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Student instructions for cases */
.student-instructions {
  .alert-warning {
    background-color: #fff3cd;
    border-color: #ffeaa7;
    color: #856404;
    border-left: 4px solid #f39c12;
    
    h4 {
      color: #856404;
      font-size: 1.1rem;
      margin-bottom: 10px;
      
      i {
        margin-right: 8px;
      }
    }
  }
}

/* Presenting symptoms styling */
.presenting-symptoms {
  h4 {
    color: #17a2b8;
    font-size: 1.2rem;
    margin-bottom: 15px;
    
    i {
      margin-right: 8px;
    }
  }
  
  .symptoms-box {
    background-color: #e3f2fd;
    border-radius: 8px;
    border-left: 4px solid #17a2b8;
    font-size: 1.05rem;
    line-height: 1.6;
  }
}

/* Enhanced Exam Completion Banner */
.exam-completion-banner {
  .completion-card {
    background: linear-gradient(135deg, #f8f9fa 0%, #ffffff 100%);
    border: none;
    border-radius: 20px;
    overflow: hidden;
    box-shadow: 0 10px 30px rgba(0,0,0,0.1);
    
    .completion-header {
      background: linear-gradient(135deg, #28a745 0%, #20c997 100%);
      padding: 30px;
      color: white;
      display: flex;
      align-items: center;
      gap: 20px;
      
      .completion-icon {
        i {
          font-size: 3rem;
          color: #ffd700;
        }
      }
      
      .completion-content {
        .completion-title {
          font-size: 1.8rem;
          font-weight: 700;
          margin-bottom: 8px;
        }
        
        .completion-subtitle {
          font-size: 1.1rem;
          opacity: 0.9;
          margin: 0;
        }
      }
    }
    
    .completion-stats {
      padding: 25px 30px;
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
      gap: 20px;
      background: white;
      
      .stat-item {
        text-align: center;
        padding: 15px;
        border-radius: 12px;
        background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
        
        .stat-value {
          font-size: 2rem;
          font-weight: 700;
          color: #28a745;
          margin-bottom: 5px;
        }
        
        .stat-label {
          font-size: 0.9rem;
          color: #6c757d;
          font-weight: 600;
          text-transform: uppercase;
          letter-spacing: 0.5px;
        }
      }
    }
    
    .completion-footer {
      padding: 25px 30px;
      background: #f8f9fa;
      text-align: center;
      
      .performance-indicator {
        display: inline-flex;
        align-items: center;
        gap: 8px;
        padding: 12px 20px;
        border-radius: 25px;
        font-weight: 600;
        font-size: 1rem;
        margin-bottom: 15px;
        
        &.performance-excellent {
          background: linear-gradient(135deg, #28a745 0%, #20c997 100%);
          color: white;
        }
        
        &.performance-good {
          background: linear-gradient(135deg, #17a2b8 0%, #20c997 100%);
          color: white;
        }
        
        &.performance-satisfactory {
          background: linear-gradient(135deg, #ffc107 0%, #fd7e14 100%);
          color: #212529;
        }
        
        &.performance-needs-improvement {
          background: linear-gradient(135deg, #fd7e14 0%, #dc3545 100%);
          color: white;
        }
        
        &.performance-poor {
          background: linear-gradient(135deg, #dc3545 0%, #6f42c1 100%);
          color: white;
        }
        
        i {
          font-size: 1.2rem;
        }
      }
      
      .review-note {
        color: #6c757d;
        font-size: 1rem;
        margin: 0;
        line-height: 1.5;
      }
    }
  }
}

/* Exam instructions styling */
.exam-instructions {
  margin-top: 30px;
  margin-bottom: 20px;
  
  h3 {
    color: #0056b3;
    font-size: 1.4rem;
    margin-bottom: 20px;
    
    i {
      margin-right: 8px;
    }
  }
  
  ol {
    padding-left: 20px;
    
    > li {
      margin-bottom: 15px;
      line-height: 1.6;
      
      strong {
        color: #333;
      }
      
      ul {
        margin-top: 8px;
        margin-bottom: 0;
        padding-left: 20px;
        
        li {
          margin-bottom: 5px;
          list-style-type: disc;
        }
      }
    }
  }
  
  &.alert-info {
    background-color: #e7f3ff;
    border-color: #b8daff;
    padding: 25px;
    border-radius: 8px;
  }
  
  .alert-warning {
    background-color: #fff3cd;
    border-color: #ffeaa7;
    color: #856404;
    padding: 15px;
    border-radius: 6px;
    
    i {
      margin-right: 5px;
    }
  }
}

/* Exam info card styling */
.exam-info {
  padding: 40px 0;
  
  .exam-card {
    background: white;
    border-radius: 15px;
    box-shadow: 0 5px 25px rgba(0,0,0,0.1);
    padding: 40px;
    
    h2 {
      color: var(--primary-color);
      margin-bottom: 20px;
    }
    
    .lead {
      font-size: 1.1rem;
      color: #666;
      margin-bottom: 30px;
    }
  }
  
  .exam-details {
    display: flex;
    gap: 30px;
    margin-bottom: 30px;
    
    .detail-item {
      display: flex;
      gap: 15px;
      
      .detail-icon {
        width: 50px;
        height: 50px;
        background: var(--primary-color);
        color: white;
        border-radius: 10px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 1.5rem;
      }
      
      .detail-content {
        h4 {
          margin: 0;
          font-size: 1rem;
          color: #666;
        }
        
        p {
          margin: 5px 0 0 0;
          font-size: 1.1rem;
          font-weight: 600;
          color: #333;
        }
      }
    }
  }
}

</style>
