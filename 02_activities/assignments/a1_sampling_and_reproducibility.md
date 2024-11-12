# ASSIGNMENT: Sampling and Reproducibility in Python

Read the blog post [Contact tracing can give a biased sample of COVID-19 cases](https://andrewwhitby.com/2020/11/24/contact-tracing-biased/) by Andrew Whitby to understand the context and motivation behind the simulation model we will be examining.

## Question 1
Examine the code in `whitby_covid_tracing.py`. Identify all stages at which sampling is occurring in the model. Describe in words the sampling procedure, referencing the functions used, sample size, sampling frame, any underlying distributions involved, and how these relate to the procedure outlined in the blog post.

## Question 2
Run the Python script file called whitby_covid_tracing.py as is and compare the results to the graphs in the original blog post. Does this code appear to reproduce the graphs from the original blog post?

## Question 3
Modify the number of repetitions in the simulation to 1000 (from the original 50000). Run the script multiple times and observe the outputted graphs. Comment on the reproducibility of the results.

## Question 4
Alter the code so that it is reproducible. Describe the changes you made to the code and how they affected the reproducibility of the script file. The output does not need to match Whitby’s original blogpost/graphs, it just needs to produce the same output when run multiple times
    
   
    

# Author: LAURA BARBOSA


## Question 1 

In this code, sampling occurs at different stages, and each stage models a part of the contact tracing scenario from the blog. Here, I'll walk through each step, highlighting the functions, sample sizes, sampling frames, and distributions.

### 1. Creating the Event Population
- **Function**: `simulate_event()`
- **Sampling Frame**: This step involves creating a DataFrame (`ppl`) that represents everyone attending the events—200 people at weddings and 800 people at brunches. This is the base population for the rest of the sampling.
- **Details**:
  - Everyone is assigned to either a wedding or brunch.
  - There’s no randomness in deciding the number of attendees—200 people are assigned to weddings and 800 to brunches.

### 2. Random Infection Sampling
- **Procedure**: **Randomly Infecting People**
- **Function**: `np.random.choice()`
- **Sample Size**: 10% of attendees (which is 100 people).
- **Sampling Frame**: The entire group of 1000 people at events.
- **Distribution**: A uniform distribution is used—each person has an equal 10% chance (`ATTACK_RATE`) of being infected, regardless of the event type.
- **Relation to Blog Post**: This simulates the "true proportion" of infections, similar to the graph showing the actual distribution of infections across different events.

### 3. Primary Contact Tracing
- **Procedure**: **Determining Who Gets Traced**
- **Function**: `np.random.rand(sum(ppl['infected'])) < TRACE_SUCCESS`
- **Sample Size**: 20% of infected people (`TRACE_SUCCESS = 0.20`).
- **Sampling Frame**: Only those who were marked as infected.
- **Distribution**: A Bernoulli trial is used—each infected person has a 20% chance of being traced.
- **Relation to Blog Post**: This models the "primary contact tracing" stage, where only some of the infected individuals get traced due to resource and recall limitations. This introduces the first bias, since not all infected people are traced.

### 4. Secondary Contact Tracing
- **Procedure**: **Tracing Events with Multiple Cases**
- **Function**: `event_trace_counts[event_trace_counts >= SECONDARY_TRACE_THRESHOLD].index`
- **Sample Size**: Varies based on how many events have two or more people traced (`SECONDARY_TRACE_THRESHOLD = 2`).
- **Sampling Frame**: Events that already have at least two traced attendees. Once this condition is met, every infected person at that event is traced.
- **Details**:
  - This is conditional sampling—secondary tracing kicks in only for events with at least two traced people.
- **Relation to Blog Post**: This represents the "secondary contact tracing" stage, where additional efforts are made to trace everyone at events like weddings. This step biases the sample by over-representing large, easily traceable events.

### 5. Running the Simulation Multiple Times
- **Procedure**: **Simulation Loop**
- **Function**: `[simulate_event(m) for m in range(50000)]`
- **Sample Size**: 50,000 iterations.
- **Sampling Frame**: Each iteration represents a new run of the infection and tracing processes.
- **Distribution**: This allows for observing overall trends by running the entire simulation multiple times.
- **Relation to Blog Post**: By repeating the simulation, we see the systematic bias in action—observing how the true versus traced infection rates differ, much like the red and blue histograms in the blog post.

### Summary of the Sampling Stages and Their Impact
- **Creating Event Population**: Establishes a fixed group of attendees for weddings and brunches—there’s no randomness here.
- **Infection Sampling (`np.random.choice`)**: Randomly selects 10% of individuals to be infected. This is a uniform sampling.
- **Primary Contact Tracing (`np.random.rand`)**: Randomly traces 20% of infected individuals, which biases the results since not all infected people are traced.
- **Secondary Contact Tracing (`event_trace_counts`)**: Focuses on events with multiple initial traces, which tends to favor larger gatherings like weddings and leads to overrepresentation.
- **Repeated Sampling (Simulation Loop)**: Runs the process 50,000 times to observe the variability in outcomes and visualize both true and observed infection distributions.

In overall, the sampling steps are designed to highlight the bias that happens in real-world contact tracing—especially showing how events like weddings tend to be overrepresented since they’re easier to fully trace once a few infections are linked to them. The observed (red histogram) data tends to overestimate the proportion of infections from these settings, as described in the blog post.

---
## Question 2
The Python script provided appears to replicate a similar pattern to what is shown in the graphs from the original blog post. Here's how the graphs from the script compare to those in the blog post:

1. **True Proportion of Infections (Blue Histogram)**:
   - The blue histogram in the script lines up with the blog’s version, centering around 0.2. This shows that 20% of infections were from weddings, with a tall, narrow shape and little variation—just like in the original.

2. **Observed Proportion of Traced Infections (Red Histogram)**:
   - The red histogram also matches the blog post’s shape, but it shifts more to the right, often around 0.4 or higher. This highlights how weddings end up overrepresented because of biased tracing.
  
In conclussion, the code captures what the blog post describes—the true infection proportion is consistently lower than what the tracing process suggests. The observed data is skewed, making weddings look like a bigger source of infections than they really are, which is clearly reflected in the red histogram’s broader and right-shifted distribution.

## Question 3 

When the number of iterations is reduced to 1000 from 50000, it has a noticeable effect on reproducibility:

1. **More Variability**: With only 1000iterations, the results vary a lot more between runs. Each time the script is run, the blue and red histograms can look quite different.

2. **Less Consistency**: The "True" and "Observed" proportions (blue and red histograms) won't always match up as well from run to run, making it harder to see consistent patterns.

3. **Jagged Distributions**: Fewer iterations mean the histograms end up looking more jagged and uneven, compared to the smoother graphs you get with 50000 iterations.

**Summary**: Using 1000 iterations reduces reproducibility, making it tougher to see clear trends. This highlights why a larger number of iterations is needed for more reliable results.

## Question 4



To make the code reproducible, I added a random seed. Setting a seed ensures that the random number generation is consistent every time the code is run, resulting in the same output. Below are the changes made:

**Code Modifications:**

- **Import random Module:**

Added the random module from Python's standard library to ensure consistent behavior across all random processes.
```python
Copy code
import random
```

- **Set Random Seeds:**

I set seeds for both NumPy and Python's random library to make all random operations deterministic. This ensures consistent results each time the script is run.

```python
Copy code
random.seed(42)
np.random.seed(42)
```

```python
# Updated Code:

# Import necessary libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import random  # Newly added import for Python's random library

# Note: Suppressing FutureWarnings to maintain a clean output. This is specifically to ignore warnings about
# deprecated features in the libraries we're using (e.g., 'use_inf_as_na' option in Pandas, used by Seaborn),
# which we currently have no direct control over. This action is taken to ensure that our output remains
# focused on relevant information, acknowledging that we rely on external library updates to fully resolve
# these deprecations. Always consider reviewing and removing this suppression after significant library updates.
import warnings
warnings.simplefilter(action='ignore', category=FutureWarning)

# Set seeds for reproducibility
random.seed(42)
np.random.seed(42)

# Constants representing the parameters of the model
ATTACK_RATE = 0.10
TRACE_SUCCESS = 0.20
SECONDARY_TRACE_THRESHOLD = 2

def simulate_event(m):
    """
    Simulates the infection and tracing process for a series of events.
    
    This function creates a DataFrame representing individuals attending weddings and brunches,
    infects a subset of them based on the ATTACK_RATE, performs primary and secondary contact tracing,
    and calculates the proportions of infections and traced cases that are attributed to weddings.
    
    Parameters:
    - m: Dummy parameter for iteration purposes.
    
    Returns:
    - A tuple containing the proportion of infections and the proportion of traced cases
      that are attributed to weddings.
    """
    # Create DataFrame for people at events with initial infection and traced status
    events = ['wedding'] * 200 + ['brunch'] * 800
    ppl = pd.DataFrame({
        'event': events,
        'infected': False,
        'traced': np.nan  # Initially setting traced status as NaN
    })

    # Explicitly set 'traced' column to nullable boolean type
    ppl['traced'] = ppl['traced'].astype(pd.BooleanDtype())

    # Infect a random subset of people
    infected_indices = np.random.choice(ppl.index, size=int(len(ppl) * ATTACK_RATE), replace=False)
    ppl.loc[infected_indices, 'infected'] = True

    # Primary contact tracing: randomly decide which infected people get traced
    ppl.loc[ppl['infected'], 'traced'] = np.random.rand(sum(ppl['infected'])) < TRACE_SUCCESS

    # Secondary contact tracing based on event attendance
    event_trace_counts = ppl[ppl['traced'] == True]['event'].value_counts()
    events_traced = event_trace_counts[event_trace_counts >= SECONDARY_TRACE_THRESHOLD].index
    ppl.loc[ppl['event'].isin(events_traced) & ppl['infected'], 'traced'] = True

    # Calculate proportions of infections and traces attributed to each event type
    ppl['event_type'] = ppl['event'].str[0]  # 'w' for wedding, 'b' for brunch
    wedding_infections = sum(ppl['infected'] & (ppl['event_type'] == 'w'))
    brunch_infections = sum(ppl['infected'] & (ppl['event_type'] == 'b'))
    p_wedding_infections = wedding_infections / (wedding_infections + brunch_infections)

    wedding_traces = sum(ppl['infected'] & ppl['traced'] & (ppl['event_type'] == 'w'))
    brunch_traces = sum(ppl['infected'] & ppl['traced'] & (ppl['event_type'] == 'b'))
    p_wedding_traces = wedding_traces / (wedding_traces + brunch_traces)

    return p_wedding_infections, p_wedding_traces

# Run the simulation 1000 times
results = [simulate_event(m) for m in range(1000)]
props_df = pd.DataFrame(results, columns=["Infections", "Traces"])

# Plotting the results
plt.figure(figsize=(10, 6))
sns.histplot(props_df['Infections'], color="blue", alpha=0.75, binwidth=0.05, kde=False, label='Infections from Weddings')
sns.histplot(props_df['Traces'], color="red", alpha=0.75, binwidth=0.05, kde=False, label='Traced to Weddings')
plt.xlabel("Proportion of cases")
plt.ylabel("Frequency")
plt.title("Impact of Contact Tracing on Perceived Infection Sources")
plt.legend()
plt.tight_layout()
plt.show()
```

![alt text](image.png)


### Impact in reproducibility:

- **Consistent Results Every Time:** By setting these seeds, the random elements (like who gets infected or traced) wil behave the same way every time the script runs. This makes sure the output is always identical, regardless of how many times the code is run.
- **Less Variability:** Before adding the seeds, each run would give slightly different results due to the randomness, which could make it hard to interpret or compare. However, the plot looks the same each time, making it easier to provide consistent conclusions.

## Criteria

|Criteria|Complete|Incomplete|
|--------|----|----|
|Altercation of the code|The code changes made, made it reproducible.|The code is still not reproducible.|
|Description of changes|The author explained the reasonings for the changes made well.|The author did not explain the reasonings for the changes made well.|

## Submission Information

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

### Submission Parameters:
* Submission Due Date: `HH:MM AM/PM - DD/MM/YYYY`
* The branch name for your repo should be: `sampling-and-reproducibility`
* What to submit for this assignment:
    * This markdown file (sampling_and_reproducibility.md) should be populated.
    * The `whitby_covid_tracing.py` should be changed.
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sampling/pull/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [ ] Create a branch called `sampling-and-reproducibility`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via our Slack at `#cohort-3-help`. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.
