# VR
using System;
using System.Collections;
using System.Collections.Generic;
using System.Text;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.Windows.Speech;

public class Posemaker : MonoBehaviour
{
    [SerializeField] Transform RightArm, FinalPoint;
    [SerializeField] bool check, _walking, _talking;
    [SerializeField] GameObject humanoid;
    [SerializeField] Animator humanoidAnimator;
    [SerializeField] float distance,minRadius = 1.2f;
    [SerializeField] GameObject textCanvas,textPanel;
    private Vector3  objectDirection,objectLocation;
    [SerializeField] Button HelpButton;
    [SerializeField]
    private Text m_Hypotheses;
    private string desiredIntent = "Helping in Switching on Moniter";

    [SerializeField]
    private Text m_Recognitions;
    private DictationRecognizer m_DictationRecognizer;

    // Start is called before the first frame update
    void Start()
    {
        humanoid = GameObject.FindGameObjectWithTag("Player");
        humanoidAnimator = humanoid.transform.GetChild(0).transform.GetComponent<Animator>();
        RightArm = GameObject.Find("RightArm").transform;
        objectDirection = FinalPoint.transform.position - RightArm.transform.position;
        textCanvas = GameObject.FindGameObjectWithTag("MainARCanvas");
        HelpButton = GameObject.FindGameObjectWithTag("Help").GetComponent<Button>();
        textPanel = GameObject.FindGameObjectWithTag("MainARCanvas").transform.GetChild(0).gameObject;
        


        m_DictationRecognizer = new DictationRecognizer();

        m_DictationRecognizer.DictationResult += (text, confidence) =>
        {
            Debug.LogFormat("Dictation result: {0}", text);
             m_Recognitions.text = text;
        };

        m_DictationRecognizer.DictationHypothesis += (text) =>
        {
            Debug.LogFormat("Dictation hypothesis: {0}", text);
            m_Hypotheses.text = text;
        };

        m_DictationRecognizer.Start();


    }
  
    // Update is called once per frame
    void Update()
    {
        if (m_Recognitions.text.Contains(desiredIntent))
        {
            IntentExecution();
        }
    }
    // Humanoid Intent Execution in 3D and AR Space
    public void IntentExecution()
    {
        objectLocation = FinalPoint.transform.position - humanoid.transform.position;
        distance = Vector3.Distance(gameObject.transform.position, humanoid.transform.position);
        if (check)
        {
            check = false;
            RightArm.transform.up = objectDirection;
          //  humanoid.transform.forward= objectDirection;
        }
        if (distance > minRadius & _walking)
        {
            humanoid.transform.Translate(objectLocation.normalized * 1f * Time.deltaTime);
        }
        else if (distance < minRadius)
        {
            humanoidAnimator.SetTrigger("Idle");
            humanoidAnimator.ResetTrigger("Walking");
            StartCoroutine(PointFinger());
        }
    }
    IEnumerator PointFinger()
    {
        yield return new WaitForSeconds(1.5f);
        humanoidAnimator.enabled = false;
        check = true;
        textPanel.SetActive(true);
    }
    public void walkAnim()
    {
        humanoidAnimator.SetTrigger("Walking");
        _walking = true;
    }
    public void checkFunc() {
        if (!check)
        {
            check = true;
        }
        else if(check)
        {
            check = false;
        }
    }
}

